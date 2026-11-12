# Fitness Coach — MasterView v1

A personal fitness dashboard frontend, made of a single static HTML file.

## Hosting (free)

**Cloudflare Pages direct upload**: Open https://dash.cloudflare.com/ → Workers & Pages → Create application → Drag and drop your files. Upload `fitness-masterview-site.zip`. The website will be served at `https://<your-project>.pages.dev`.

**GitHub Pages**: Push `index.html` to a public repository and enable Pages in Settings. Only static HTML, CSS and client JavaScript appear in the public repo; do not add secrets or private health data.

## First-time authorization

1. Supabase Dashboard → Authentication → URL Configuration → add the final public dashboard URL under Redirect URLs (including optional trailing slash if needed).
2. Open the dashboard and click `Zaloguj`. Authenticate using the email link / OTP.
3. Signing in does not grant access to Garmin data. An administrator must approve the resulting Supabase Auth user ID in the protected table `public.fitness_dashboard_access`. Keep the allowlist empty until the intended user is verified. Example SQL (after verifying the user ID):

```sql
insert into public.fitness_dashboard_access (user_id, note)
values ('<VERIFIED_AUTH_USER_ID>', 'Athlete owner')
on conflict (user_id) do nothing;
```

4. Reload the page after approval. The client calls `https://ajleirrusjtxsalisfqq.supabase.co/functions/v1/fitness-dashboard/api` with the user's authenticated JWT. The API checks the allowlist server-side and returns only authorized data.

## Architecture / security

- Supabase Edge Function `fitness-dashboard` v2 is an authenticated **API only**. Supabase's default Edge domain rewrites HTML responses to text/plain, so it is not the frontend hosting location.
- The bundled `index.html` embeds only a **publishable** Supabase key, which is expected to be publicly visible. It does not contain the Garmin API key, Supabase secret keys, or any real health data.
- Real activities, sleep and HRV are returned by an API gated by Supabase Auth and the explicit `fitness_dashboard_access` allowlist. Database RLS is enabled on the underlying tables.
- The default dashboard shows **synthetic demo data**, clearly labeled. Never mistake these for the athlete's measurements.
- Garmin sessions and manually reported workouts are shown together; linked Garmin activity IDs prevent double-counting.
- Submitting a daily subjective report is optional.

## Updating the site

Edit `index.html` locally and re-deploy the folder/ZIP. The API backend and database remain in Supabase.
