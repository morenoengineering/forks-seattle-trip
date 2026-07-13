# Forks → Seattle Trip Planner

Self-contained single-page trip planner (itinerary, packing list, reservations,
document vault, emergency/health info). No backend — all saved data lives in
each visitor's own browser (localStorage).

## Deploy to GitLab Pages

1. Create a new project on gitlab.com (or your self-hosted instance).
2. From this folder, run:
   ```
   git init
   git add .
   git commit -m "Trip planner site"
   git branch -M main
   git remote add origin git@gitlab.com:<your-username>/<your-project>.git
   git push -u origin main
   ```
3. In your GitLab project: **CI/CD → Pipelines** — wait for it to go green
   (should take well under a minute, there's no real build step).
4. Your site is live at:
   ```
   https://<your-username>.gitlab.io/<your-project>
   ```
   Check **Settings → Pages** in the project if you don't see the URL yet —
   first deploys sometimes take a minute or two to appear there.

## Updating the site later

Edit `public/index.html`, then:
```
git add .
git commit -m "Update trip planner"
git push
```
The pipeline redeploys automatically.

## Notes

- Every visitor's checked-off packing items, reservation statuses, and vault
  entries are local to *their own* device/browser. Nothing syncs between
  people — this is a shared *view*, not a shared *database*.
- The `.gitlab-ci.yml` here is intentionally minimal since there's nothing to
  compile — it just tells GitLab Pages to publish the `public/` folder as-is.

## Password protection

The site is gated behind a password ("maya-eats-quesadilla") using client-side
AES-256-GCM encryption (key derived via PBKDF2, 250,000 iterations). The actual
trip content — flight/car confirmations, addresses, health info, everything —
is stored in the page only as ciphertext. It's decrypted in-browser only after
the correct password is entered, using the Web Crypto API.

**What this protects against:** someone stumbling on the link, a search engine
indexing it, or a casual "view source" — none of them see anything but
unreadable ciphertext.

**What this does NOT protect against:** someone determined to brute-force the
password offline (they'd need to copy the ciphertext and try passwords locally
with no rate limit), or anyone who already knows the password sharing it
further. There's also no "forgot password" recovery — if it's mistyped, the
only fix is knowing the real one.

This is appropriate for keeping the page private from the general internet,
not for defending genuinely sensitive data (e.g. don't rely on this alone if
you ever add anything like payment info).

### Changing the password later

The password is baked into the encrypted payload at build time — there's no
way to change it by editing the HTML directly. Ask for a rebuilt file with a
new password if needed.
