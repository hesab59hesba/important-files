# Supabase Restore Runbook — Menasat Benaa (2026-11-11)

> What actually happened when restoring the live `wvsopwtelslsaytyqzgh` database into a fresh
> Supabase project (`olzwucserumbjeadiljm`), the 7 problems hit, and the exact fixes.
> Read this **before** attempting a restore. Contains no secrets.

---

## 0. Artifacts (from the 2026-11-11 backup)

| Artifact | Size | Purpose |
|---|---|---|
| `menasat_backup_full_2026-11-11.zip` | 2.34 GB | everything below in one zip |
| `roles.sql` / `schema.sql` / `data.sql` | 358 B / 419 KB / 31.7 MB | Supabase-CLI 3-part dump (restore target: another Supabase project) |
| `db_full_public_auth_storage.dump` | 6.6 MB | `pg_dump -Fc` of schemas `public`+`auth`+`storage` — **use this one for the data restore** |
| `storage/` (3,843 files, 2.2 GB) | — | bucket contents; upload separately only if quota allows |

## 1. Connectivity: direct DB host is IPv6-only

- `db.<ref>.supabase.co` resolves **only to IPv6**. GitHub Codespaces (and most CI) have no IPv6 → direct connection **fails** (`Network is unreachable`).
- **Fix:** always use the **Supavisor session pooler**: `postgresql://postgres.<ref>:<pass>@aws-<n>-<region>.pooler.supabase.com:5432/postgres`
  - port **5432** (session mode) — port 6543 (transaction mode) does **not** work with pg_dump/pg_restore.
  - **Session mode required** for `pg_restore` and `ALTER`-heavy work.
- Region: visible in Dashboard → Settings → General, or in the Session-pooler connection string. (Probing regions with a hand-rolled Postgres startup packet fails — Supavisor rejects it as `bad_startup_payload`. Don't bother; just read the string from the dashboard.)
- pg_dump/pg_restore must be **≥ server version** (server: PostgreSQL 17.x → use PGDG `postgresql-client-17`; Ubuntu noble ships client-16 which is too old for pg_dump).
- The `supabase` CLI bundles its own pg_dump 17 (pulls a Docker image on first `db dump`) — fine for the 3-part dump, but for `pg_restore` use the PGDG client.

## 2. Supabase-CLI 3-part dump: works, but has two gotchas

```
supabase db dump --db-url "$URL" -f roles.sql --role-only
supabase db dump --db-url "$URL" -f schema.sql
supabase db dump --db-url "$URL" -f data.sql --data-only --use-copy
```

- **`data.sql` DOES include `auth` + `storage` table data** (26 auth + 9 storage COPYs) — verified.
- The FK-ordering hint pg_dump prints ("consider --disable-triggers") is **misleading on modern Supabase targets** — see §4.
- `schema.sql` includes `supabase_functions.*` hook-table references → 2 harmless errors on restore ("schema supabase_functions does not exist"). Ignore.

## 3. Fresh-project permission model (the big one)

On current Supabase projects the `postgres` role is **NOT superuser**:

```
rolname  | rolsuper | rolbypassrls
postgres |    f     |      t        ← RLS bypassed, but…
supabase_auth_admin  owns auth.*     ← postgres can't ALTER/DISABLE TRIGGER there
supabase_storage_admin owns storage.*
```

Consequences:
- ❌ `pg_restore --disable-triggers` **fails**: postgres can't set `session_replication_role`, and can't disable RI/system triggers ("permission denied: RI_ConstraintTrigger_c_… is a system trigger", "must be owner of table users"). Each failure **aborts that table's COPY** (COPY is all-or-nothing).
- ❌ Direct writes to platform tables like `auth.schema_migrations` are denied — skip them.
- ✅ `postgres` has `rolbypassrls=t` → data COPYs bypass RLS fine.
- ✅ postgres **owns** everything in `public` (schema restored as postgres) → can ALTER TABLE/TRIGGER/FUNCTION there.

## 4. The data restore that actually works

Use the `-Fc` dump, data-only, **without** `--disable-triggers`, and **iterate to a fixpoint**:

```sh
pg_restore -d "$URL" --data-only --no-owner db_full_public_auth_storage.dump
# repeat until no new tables gain rows (expect ~4-7 passes)
```

Why iterations: pg_restore runs the dump's TOC in a fixed order that is **not topologically
FK-safe** for this schema (children sometimes run before their parents). Each pass fills
tables whose parents are now complete; already-restored tables abort on duplicate-key
(harmless — they stay intact). Empirically: pass 1 ≈ 45%, pass 2 ≈ 90%, passes 3-7 finish the
rest (community chain: posts → comments → comment_likes each need the previous pass).

**Before pass 1, apply §5** (trigger search_path) — otherwise community restore stalls.

## 5. Trigger functions with mutable search_path break restore

`increment_comment_count` / `notify_community_*` etc. do `UPDATE community_posts SET …`
**unqualified**. Under pg_restore the session `search_path` is empty → `relation "community_posts"
does not exist` → the COPY aborts → comments & comment_likes never restore.

**Fix (also resolves Supabase's security advisor WARN `function_search_path_mutable`):**

```sql
DO $$ DECLARE r record; n int := 0; BEGIN
  FOR r IN SELECT p.oid::regprocedure AS fn
           FROM pg_proc p JOIN pg_roles ro ON ro.oid = p.proowner
           WHERE p.pronamespace = 'public'::regnamespace AND ro.rolname = 'postgres'
             AND (p.proconfig IS NULL OR NOT EXISTS (
                   SELECT 1 FROM unnest(p.proconfig) c WHERE c LIKE 'search_path%'))
  LOOP EXECUTE format('ALTER FUNCTION %s SET search_path = public', r.fn); n := n+1; END LOOP;
  RAISE NOTICE 'pinned % functions', n;
END $$;
```

(33 functions on this project.) While restoring tables with counter/notify triggers, also:

```sql
ALTER TABLE public.community_comments      DISABLE TRIGGER USER;  -- postgres owns it → allowed
ALTER TABLE public.community_comment_likes DISABLE TRIGGER USER;
-- … restore …
ALTER TABLE public.community_comments      ENABLE TRIGGER USER;
ALTER TABLE public.community_comment_likes ENABLE TRIGGER USER;
```

## 6. Trigger side effects during restore → clean up afterwards

Triggers **fire during data COPYs** (they can't be system-disabled). Two artifacts appeared:

1. **Duplicate notifications**: `notify_community_like` / `upsert_message_notification` fired when
   their source tables were restored, inserting notification rows that the `notifications` COPY
   later also brought → `notifications` ended +160 over backup state.
   **Fix:** delete the restore-window rows — restore artifacts have `created_at` inside the restore
   time window; backup rows are ≤ backup time:
   ```sql
   DELETE FROM public.notifications WHERE created_at >= '<restore-window-start>';
   ```
2. **Counter drift** (increments fired on top of COPYed counters):
   ```sql
   UPDATE public.community_posts p SET
     comments_count = (SELECT count(*) FROM public.community_comments c WHERE c.post_id=p.id),
     likes_count    = (SELECT count(*) FROM public.community_post_likes l WHERE l.post_id=p.id),
     views_count    = (SELECT count(*) FROM public.community_post_views v WHERE v.post_id=p.id);
   UPDATE public.community_comments c SET
     likes_count = (SELECT count(*) FROM public.community_comment_likes cl WHERE cl.comment_id=c.id);
   UPDATE public.professional_groups g SET member_count =
     (SELECT count(*) FROM public.group_members m WHERE m.group_id=g.id);
   UPDATE public.in_app_announcements a SET open_count =
     (SELECT count(*) FROM public.announcement_opens o WHERE o.announcement_id=a.id);
   ```

## 7. Sequences don't advance through COPY

Only two serial sequences exist (`app_performance_metrics_id_seq`, `admin_audit_logs_id_seq`):

```sql
SELECT setval('public.app_performance_metrics_id_seq', COALESCE((SELECT MAX(id) FROM public.app_performance_metrics),1));
SELECT setval('public.admin_audit_logs_id_seq',       COALESCE((SELECT MAX(id) FROM public.admin_audit_logs),1));
```

## 8. Verification checklist (live vs restored, all 63 tables)

```sql
-- run per table on both sides, then diff:
SELECT '<table>', count(*) FROM public.<table> UNION ALL …
-- expect: EXACT match. Live will drift afterwards (it's a live system) — that's healthy.
```
2026-11-11 result: 62/63 exact; `notifications` = backup-time count (live had gained 1 row since).
Also spot-check **contents**, not just counts (profiles `full_name`/`role`/phone prefixes),
because `handle_new_user` fires during `auth.users` COPY — if it ran before the `profiles` COPY
landed, you'd get skeleton rows. (It didn't — but check.)

## 9. Storage buckets: metadata ≠ files

- `storage.buckets` + `storage.objects` rows restore with the data (16 buckets, 3,843 objects) —
  the app then *knows* about every file.
- **File contents are NOT in any dump** — they must be re-uploaded (free plan quota is 1 GB; the
  full set is 2.2 GB).
- **Image URLs in the data are absolute and point at the ORIGINAL project host**
  (`https://wvsopwtelslsaytyqzgh.supabase.co/storage/v1/object/public/…`). After restore into a new
  project, the app keeps loading images from the **old** project's public CDN — by design of the
  schema, not a bug. Options: leave as-is (fidelity) / rewrite URLs to the new ref / upload a subset.
- Private buckets (`subscription-receipts`, `verification-documents`) need the secret key to read.

## 10. What silently does NOT transfer

| Thing | Why | What to do |
|---|---|---|
| Google OAuth sign-in | redirect URLs configured per-project/domain | reconfigure in tmp dashboard if needed |
| Edge function `push_notifications` | functions aren't in DB dumps | `supabase functions deploy` (needs CLI link) |
| pg_cron jobs (`bourse-morning/evening`, `disable-expired-maintenance`) | cron.schedule isn't in dumps | re-run the mirrored migrations' scheduling part (files in `supabase/migrations/`) |
| Auth SMTP/providers config, JWT signing keys | dashboard settings | copy manually if you want identical behavior |
| `auth.refresh_tokens`/`sessions` may partially fail on FK to sessions — non-blocking | users just log in again |

## 11. End-to-end recipe (next time, condensed)

```sh
# 1. dump (old project, session pooler URL)
supabase db dump --db-url "$OLD" -f roles.sql --role-only
supabase db dump --db-url "$OLD" -f schema.sql
supabase db dump --db-url "$OLD" -f data.sql --data-only --use-copy   # optional
pg_dump "$OLD" -Fc --no-owner --schema=public --schema=auth --schema=storage -f full.dump

# 2. restore into new project (session pooler URL)
psql "$NEW" -f roles.sql                                   # ignore 1 harmless error
psql "$NEW" -f schema.sql                                  # expect 2 harmless supabase_functions errors
# pin search_path (§5) — MUST precede data
pg_restore -d "$NEW" --data-only --no-owner full.dump      # repeat 4-7× until fixpoint
# clean notifications window (§6.1), recompute counters (§6.2), setval sequences (§7)

# 3. verify (§8): all-tables diff vs old project + content spot-checks
# 4. app: point .env at new URL/anon key → flutter run -d web-server --release
# 5. storage contents: upload only if quota allows; private buckets need secret key
```

---

## 12. Offsite copy to Backblaze B2 (large-file API)

For files >5 GB use parts; even ~2.3 GB benefits from the large-file flow:

```python
b2_authorize_account (Basic keyID:applicationKey) → apiUrl, authorizationToken, allowed
b2_start_large_file {bucketId, fileName, contentType, fileInfo} → fileId
b2_get_upload_part_url {fileId} → uploadUrl + token
# POST each part with X-Bz-Part-Number, X-Bz-Content-Sha1, Content-Length
b2_finish_large_file {fileId, partSha1Array}
```

**Gotcha hit:** reusing ONE `b2_get_upload_part_url` URL/token for parallel workers produced
sporadic `HTTP 400` (empty body) on ~1/3 of requests. **Fix: fetch a FRESH part-upload URL per
part** (the call is cheap) — zero failures afterwards. Parts may upload out of order; on resume,
`b2_list_parts` gives the authoritative list for `b2_finish_large_file`. Verify final
`contentLength` equals the source size.
