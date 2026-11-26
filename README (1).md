# Sure In Bayane (SIB) — rebuilt project files

This package preserves the current SIB main site and adds a working research-group workspace.

## Files

- `index.html` — main SIB site: account creation/login, Google OAuth, profile, researcher directory, research submission/review view, and multi-source literature search.
- `admin.html` — same current application entry point. The review screen remains hidden unless the signed-in user is listed in `public.admins`; actual permissions must also be enforced by Supabase RLS.
- `research-groups.html` — research workspaces: group creation, private/public visibility, invitations and acceptance, realtime discussions, shared file storage, shared reference library, meetings, tasks, and activity log.
- `supabase-migration.sql` — database tables, indexes, RLS policies, private Storage bucket, invitation notifications, and Realtime publication setup for group features.
- `robots.txt` and `sitemap.xml` — crawler configuration. Private group and admin screens are excluded from indexing.

## Deployment order

1. Back up the current Supabase database and Storage settings.
2. Open Supabase → SQL Editor and run `supabase-migration.sql`.
3. In Supabase → Authentication → URL Configuration, set the production Site URL and add the exact GitHub Pages URL (and local development URL if used) to Redirect URLs.
4. In Authentication → Providers, enable Google only after configuring the Google OAuth client and callback URL shown by Supabase.
5. Confirm the `avatars` bucket and its existing policies still work for profile pictures. The migration creates a separate private `group-files` bucket.
6. Upload/replace the website files in the same GitHub Pages project root. Keep filenames exactly `index.html`, `admin.html`, `research-groups.html`, `robots.txt`, and `sitemap.xml`.
7. Test with two ordinary accounts: create a group with account A, invite account B using B's profile UUID, accept from B, then test discussion, files, references, meetings, and tasks.
8. Test admin moderation with a designated admin account and verify that a normal account cannot approve or delete other users' research.

## Configuration and secrets

The existing project URL and publishable/anon browser key are kept in the front-end because these are designed to be public. Never put a Supabase `service_role` key, OAuth client secret, database password, or private API key in HTML, JavaScript, GitHub Pages, or this repository. Enable RLS on every user-data table. Do not treat hiding an admin button as security.

The frontend reads optional overrides before creating the Supabase client:

```html
<script>
  window.BAYANE_SUPABASE_URL = "https://YOUR_PROJECT.supabase.co";
  window.BAYANE_SUPABASE_ANON_KEY = "YOUR_PUBLISHABLE_OR_ANON_KEY";
</script>
```

Place this before the Supabase client script only if deploying to a different Supabase project. For the existing project, no override is needed.

## Important compatibility notes

- The migration is designed to extend the existing SIB schema (`profiles`, `researches`, and `admins`). It does not recreate or replace those tables.
- Review any pre-existing policies on `researches`, `profiles`, `admins`, and `avatars` before production deployment. PostgreSQL permissive policies are additive; an old overly-broad policy can weaken a new policy.
- Group invitations use a researcher's Supabase auth UUID. The app intentionally does not expose private email addresses in the directory.
- Shared group files are private and are opened using short-lived signed URLs. The bucket limit is 15 MB per file and allows common research document formats.
- A public group can be discovered in the group list only if its read policy is enabled, but membership is still required to use the private collaboration tools.
- The notifications table records invitations. The group page also displays pending invitations so users can accept or decline them.

## Suggested next upgrades

1. ORCID OAuth verification and researcher identity badges.
2. Saved papers and personal reading lists, with BibTeX/RIS export.
3. Research-project milestones, protocol checklist, and progress timeline.
4. Email digests for invitations, task deadlines, and upcoming meetings.
5. Audit export and admin moderation queue with reasoned decisions.

These are roadmap ideas, not claimed as already implemented in this package.
