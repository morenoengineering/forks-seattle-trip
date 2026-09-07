# Faroe Islands Trip

Self-contained single-page trip guide for our Faroe Islands trip, Sep 6–11 2026
(William, Victoria, Maya & Rhema). Itinerary, map, flight/car/house logistics,
notes for travelling with the twins, and a practical Faroes primer.

No backend. The packing checklist is saved in each visitor's own browser
(localStorage) — nothing syncs between people or devices.

## What's in it

Six tabs:

| Tab | Contents |
| --- | --- |
| **Now** | A live "what's happening" card driven by the device clock, three time-zone clocks (Baltimore / Reykjavík / Faroes), pre-departure checklist, and what's still unconfirmed |
| **Days** | Day-by-day plan, Night 0 (the overnight flight) through Day 5 (departure), with flex notes and safety flags |
| **Map** | Stylized map of Vágar / Streymoy / Eysturoy with tappable numbered stops, plus drive times from Vestmanna |
| **Logistics** | Icelandair flights both ways, the Enterprise rental, and the Airbnb — address, lockbox, Wi-Fi, host directions, house manual |
| **Twins** | Persistent packing checklist, plus sleep, food and outdoor-safety notes |
| **Faroes** | Emergency numbers, driving and tunnel tolls, money, weather, and local practicalities |

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

## Notes

- GitHub Pages sites are publicly reachable regardless of repo visibility
  (on free/personal plans there's no built-in auth wall) — the sign-in
  screen in `public/index.html` is what actually gates the content, not repo
  privacy.
- The page also sends `noindex, nofollow`, so search engines shouldn't list it.

## Sign-in: email allowlist + passcode

The site is gated behind two checks, both client-side (no backend, no
accounts):

1. **Email allowlist** — the visitor's email is checked against a hardcoded
   list in `public/index.html` (`APPROVED_EMAILS`). This is a plain string
   match, not an authentication factor — it doesn't verify the visitor
   actually owns that email address. It's a courtesy check to keep casual
   visitors out and to make clear who the trip content is meant for.
2. **Passcode** — client-side AES-256-GCM encryption (key derived via
   PBKDF2-HMAC-SHA256, 250,000 iterations). The actual trip content —
   flight/car confirmations, the house address, the lockbox and Wi-Fi
   details, everything — is stored in the page only as ciphertext, decrypted
   in-browser only after the correct passcode is entered, via the Web Crypto
   API.

Both the email and the passcode must be correct to unlock the page.

### Managing the approved email list

Edit the `APPROVED_EMAILS` array in the `<script>` block near the bottom of
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

## Updating the site later

The trip content is encrypted at build time, so it can't be edited directly in
`public/index.html` — the readable text simply isn't in there. To change it,
ask for a rebuild: the plaintext content is re-encrypted with the passcode and
the whole file regenerated. Same for changing the passcode.

The email allowlist, the `<title>`, and the lock-screen styling are all plain
text in `public/index.html` and can be edited directly.

After any edit:
```
git add .
git commit -m "Update trip site"
git push
```
The workflow redeploys automatically.
