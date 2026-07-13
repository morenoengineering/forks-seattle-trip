# Forks → Seattle Trip Planner

Self-contained single-page trip planner (itinerary, packing list, reservations,
document vault, emergency/health info). No backend — all saved data lives in
each visitor's own browser (localStorage).

## Deploy to GitHub Pages

1. Push this repo to GitHub.
2. In the repo: **Settings → Pages → Build and deployment → Source**, choose
   **GitHub Actions**.
3. The included `.github/workflows/pages.yml` workflow publishes the
   `public/` folder automatically on every push to `main`.
4. Your site is live at:
   ```
   https://<your-username>.github.io/<your-repo>
   ```
   Check the **Actions** tab for the workflow run, and **Settings → Pages**
   for the URL once it's deployed.

## Updating the site later

Edit `public/index.html`, then:
```
git add .
git commit -m "Update trip planner"
git push
```
The workflow redeploys automatically.

## Notes

- Every visitor's checked-off packing items, reservation statuses, and vault
  entries are local to *their own* device/browser. Nothing syncs between
  people — this is a shared *view*, not a shared *database*.
- GitHub Pages sites are publicly reachable regardless of repo visibility
  (on free/personal plans there's no built-in auth wall) — the sign-in
  screen in `index.html` is what actually gates the content, not repo
  privacy.

## Sign-in: email allowlist + passcode

The site is gated behind two checks, both client-side (no backend, no
accounts):

1. **Email allowlist** — the visitor's email is checked against a hardcoded
   list in `public/index.html` (`APPROVED_EMAILS`). This is a plain string
   match, not an authentication factor — it doesn't verify the visitor
   actually owns that email address. It's a courtesy check to keep casual
   visitors out and to make clear who the trip content is meant for.
2. **Passcode** — same as before: client-side AES-256-GCM encryption (key
   derived via PBKDF2, 250,000 iterations). The actual trip content —
   flight/car confirmations, addresses, health info, everything — is stored
   in the page only as ciphertext, decrypted in-browser only after the
   correct passcode is entered, via the Web Crypto API.

Both the email and the passcode must be correct to unlock the page.

### Managing the approved email list

Edit the `APPROVED_EMAILS` array near the top of the `<script>` block in
`public/index.html`, then commit and push:

```js
const APPROVED_EMAILS = [
  'willvict.moreno@gmail.com',
  'wjmoreno@alumni.cmu.edu'
];
```

Add or remove addresses (lowercase) as needed. This list is plain text in
the page source — don't rely on it to hide who's on the trip, only to gate
casual access.

The passcode itself is never stored anywhere in this repo (not in the HTML,
not in the README) — only the salt, IV, and ciphertext are. Share the
passcode with trip members out-of-band (text, in person, etc.), not via a
commit or issue in this repo.

**What this protects against:** someone stumbling on the link, a search engine
indexing it, or a casual "view source" — none of them see anything but
unreadable ciphertext.

**What this does NOT protect against:** someone determined to brute-force the
passcode offline (they'd need to copy the ciphertext and try passcodes
locally with no rate limit — the email check is just a JS `if`, trivially
bypassed by anyone editing the page's local script), or anyone who already
knows the passcode sharing it further. There's also no "forgot passcode"
recovery — if it's mistyped, the only fix is knowing the real one.

This is appropriate for keeping the page private from the general internet,
not for defending genuinely sensitive data (e.g. don't rely on this alone if
you ever add anything like payment info).

### Changing the passcode later

The passcode is baked into the encrypted payload at build time — there's no
way to change it by editing the HTML directly. Ask for a rebuilt file with a
new passcode if needed. (The email allowlist, by contrast, is plain text in
`public/index.html` and can be edited directly — see above.)
