# Waysperer — web share viewer

A single static page that renders a **read-only** Waysperer itinerary from a share token, so people without the app can open a shared trip in any browser. It calls the public `shared_trip` Supabase RPC (SECURITY DEFINER, granted to `anon`) — the same one the app's deep link uses — and shows only itinerary fields (no booking details).

- **URL shape:** `https://arvindpj007.github.io/waysperer-share/#<token>`
- The token rides the URL **fragment** (`#…`), so it never hits the server logs or PostgREST as a query param.
- Only the **publishable/anon** key is embedded (client-safe, same as the app ships). No secrets here — keep this repo public so free GitHub Pages works.

## Deploy (one-time)

GitHub Pages on a **private** repo needs a paid plan, so this lives in its own **public** repo:

1. Create a new **public** GitHub repo named exactly **`waysperer-share`** under `arvindpj007`.
2. Push this folder's `index.html` to its default branch:
   ```sh
   cd Projects/waysperer-share
   git init && git add . && git commit -m "Waysperer web share viewer"
   git branch -M main
   git remote add origin git@github.com:arvindpj007/waysperer-share.git
   git push -u origin main
   ```
3. Repo → **Settings → Pages** → Source = **Deploy from a branch**, branch = `main`, folder = `/ (root)` → Save.
4. Wait ~1 min, then open `https://arvindpj007.github.io/waysperer-share/` — it should show the "No trip link" state.

The app's Share sheet already hands out `https://arvindpj007.github.io/waysperer-share/#<token>`. If you deploy under a **different** repo name, update `_webBase` in `lib/features/share/share_export_sheet.dart` to match, then rebuild.

## Test

- `…/waysperer-share/` (no token) → "No trip link".
- `…/waysperer-share/#deadbeef` (bad token) → "This link has expired".
- A real live token from the app's Share sheet → the full read-only itinerary.

## Notes

- Revoked / expired links resolve to `null` from the RPC → the page shows the "expired" state, matching the app.
- The footer's **Open in the app** link uses the `waysperer://shared/<token>` scheme for installed users.
- No build step, no dependencies — it's one file. Supabase's REST endpoint sends permissive CORS, so the `github.io` origin can call it directly.
