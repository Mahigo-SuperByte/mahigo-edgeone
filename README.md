# Mahigo Group Admin — EdgeOne Makers build

This build is for **EdgeOne Makers Edge Functions** (the `edge-functions/` folder convention). It is not the earlier Express/Node.js server ZIP. The login endpoint is implemented on the EdgeOne runtime, so the admin frontend no longer tries to call a Node server that is not running on a static hosting deployment.

## Why the previous login failed

The earlier server package expected an Express server at `/api/health` and `/api/admin/login`. Uploading only its static HTML/JS to EdgeOne did not deploy those server routes. This build provides those routes as an Edge Function at `edge-functions/[[default]].js`, which catches `/api/...` requests.

## Deploy

1. Upload/deploy the **contents** of this folder as your EdgeOne Makers project. Keep `index.html`, `app.js`, and the `edge-functions/` folder at the project root.
2. In the project's Environment Variables / Secrets settings, add:
   - `ADMIN_USERNAME` = `admin`
   - `ADMIN_PASSWORD` = your chosen admin password (it must match what you enter on the login screen)
   - `SESSION_SECRET` = a random secret of at least 32 characters
   - `API_HOST` = `api.mahigo.com` (optional; set this only when your Kusum API hostname is configured)
3. Enable EdgeOne KV Storage, create a namespace, and bind it to this project using the variable name **`mahigo_kv`**. The function uses this binding for persistent admin data. EdgeOne's KV bindings are available to Edge Functions; the variable name is the binding name.
4. Redeploy the project after adding the functions, secrets, and KV binding.
5. Open the deployed site's `/api/health`. A healthy response is JSON with `ok: true` and `storage: "EdgeOne KV"`. If not, the error tells you which project setting is missing.
6. Open the admin homepage and sign in with the username/password configured above.

Do not place `ADMIN_PASSWORD` or `SESSION_SECRET` in `app.js`, `index.html`, or any file served to the browser. The login uses a signed, HTTP-only, Secure, SameSite=Lax cookie. Sessions expire after eight hours.

## What this Edge Functions build handles

- Health check, login, session check, logout.
- Persistent state in EdgeOne KV.
- Save/list/delete MahigoWebs project records.
- Save/list/delete Mahigo Kusum endpoint definitions.
- Import small JSON database files (750 KB limit for this function route).
- Save headquarters and organization settings.
- Serve configured JSON endpoints when requests reach the configured `API_HOST` hostname and the hostname is routed to this EdgeOne project.

## EdgeOne runtime limits / remaining deployment work

EdgeOne documents an Edge Functions request body limit of 1 MB. Consequently, this package intentionally returns a clear `501` error for Scratch `.sb3` packaging rather than pretending that it is hosted. MahigoGames packaging requires an EdgeOne Node.js Cloud Function and object/Blob storage (or another Node.js host) to run the TurboWarp Packager and store the resulting playable HTML. Large database imports also need Cloud Functions or a database service.

Likewise, saving a website record is not by itself the same as serving `slug.mahigo.com`. Configure the custom domains / wildcard DNS and implement host-based routing to the stored website content before calling a site public. The API hostname must also be attached to this project and routed for public Kusum endpoints.

The EdgeOne KV storage documentation states that KV data is eventually consistent across edge locations (up to 60 seconds); use a transactional database for high-concurrency production writes.

## Diagnostics

- `/api/health` — checks that secrets and the KV binding exist.
- `POST /api/admin/login` — checks the configured username/password and sets the session cookie.
- `GET /api/admin/me` — validates the cookie.
- `GET /api/admin/state` — checks authenticated KV access.

If `/api/health` returns 404, the Edge Function folder was not deployed at the expected root. If it returns 503, follow the message and configure the missing secrets/KV binding.
