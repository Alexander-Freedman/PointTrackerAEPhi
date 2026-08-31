# AEPhi CO26 Pearl Tracker

**Live site:** https://alexander-freedman.github.io/PointTrackerAEPhi/
*(confirm this matches your GitHub Pages settings — see [Deploying](#deploying-with-github-pages) below)*

A single-page web app for tracking Fall 2026 pearl points per CO26 member.
No password — sisters log in with their full name and birthday, which
together identify their counter. Data lives in a free Firebase Firestore
database, so it's persistent and shared across everyone's phones/laptops.
The browser remembers the login (via localStorage), so returning to the
page on the same device logs back in automatically — "Switch User" clears
it and returns to the login screen.

**Nothing about this is secure or authenticated.** Full name + birthday is
just enough that two people don't accidentally share a counter and it
isn't trivially guessable — but anyone who actually knows both can still
edit that person's counter. That's intentional: this is for casual
internal tracking, not anything official or sensitive.

## Contents

- [What's in here](#whats-in-here)
- [One-time Firebase setup](#one-time-setup-about-5-minutes)
- [Deploying with GitHub Pages](#deploying-with-github-pages)
- [How it works](#how-it-works)
- [Costs](#costs)
- [Troubleshooting](#troubleshooting)

## What's in here

- `index.html` — the entire app. No build step, no server, no
  dependencies beyond the Firebase SDK (loaded from a CDN). Open it
  directly or host it on GitHub Pages.
- Categories and point values are pulled directly from the Fall 2026
  Points Breakdown PDF for **MC26** (10 categories, 30 pearls total).

## One-time setup (about 5 minutes)

You need a free Firestore database for the data to live in. Do this once
— it's already done for the current `firebaseConfig` in `index.html`, so
skip to [Deploying](#deploying-with-github-pages) unless you're pointing
this at a new Firebase project.

1. Go to **console.firebase.google.com** and sign in with any Google account.
2. Click **Add project** → give it any name (e.g. `aephi-pearls`) →
   decline Google Analytics if asked → **Create project**.
3. In the left sidebar, click **Build → Firestore Database** →
   **Create database**.
   - Choose any location (closest to you is fine).
   - Start in **production mode** — we open it up manually next, so
     access never expires. ("Test mode" auto-locks the app after 30 days.)
4. Once created, open the **Rules** tab and replace the contents with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if true;
       }
     }
   }
   ```
   Click **Publish**. This makes it open to anyone with the site link —
   matches "nothing secure," and never expires.
5. Back in the project, click the gear icon (top left) → **Project settings**.
   Scroll to **Your apps** → click the **`</>`** (web) icon → give it any
   nickname → **Register app**. You do **not** need Firebase Hosting.
6. Copy the `firebaseConfig` object it shows you (apiKey, authDomain,
   projectId, storageBucket, messagingSenderId, appId).
7. In `index.html`, find `const firebaseConfig = { ... }` near the top of
   the `<script type="module">` block and replace it with the block you
   copied.

This config is **not a secret** — it's meant to be public in client-side
code. Access control comes from the Rules step above, not from hiding
this object.

## Deploying with GitHub Pages

1. Commit and push `index.html` (with your config in place) to this repo.
2. In the GitHub repo, go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to `Deploy from a branch`,
   pick the branch this file lives on, and folder `/ (root)`. Save.
4. GitHub publishes it at `https://<your-username>.github.io/<repo-name>/`
   within a minute or two — that's the link at the top of this file.
   Share it with the chapter.
5. Any future push to that branch redeploys automatically — no extra step.

## How it works

- Login is **full name + birthday**. Both are normalized (trimmed,
  lowercased) into one key, so it's not case-sensitive and two sisters
  with the same name still get separate counters — stored as one Firestore
  document per person (`counters/<name>_<birthday>`).
- Category buttons match the exact point values from the Fall 2026
  breakdown (e.g. Sisterhood Event = +2, Study Hours = +1, Lead a 2nd Ave
  = +2, etc).
- Each category has its own progress bar (current / target), plus an
  overall "X / 30 pearls" bar at the top.
- Tapping a point button opens a small popup with an optional note field
  (e.g. "Fall retreat" or "with Sarah") before it saves — useful for
  remembering which specific event a point was for.
- Under each category's progress bar is a running list of everything
  logged there: the activity, points, and note. Tap the × next to an
  entry to remove it (this is also how you correct a misclick — there's
  no separate "-1" button anymore).
- A "Miscellaneous" section covers non-category points (announcing,
  tabling, banner painting, etc.) that count toward the 30 total but
  don't belong to a specific pearl category, with its own log list.
- Updates sync live — if a sister edits her pearls on her phone, anyone
  else with her login loaded sees the change without refreshing.
- The browser remembers your last login (localStorage), so you don't
  have to retype it every visit on the same device. "Switch User" wipes
  that and returns to the login screen.

## Costs

Firebase's free "Spark" tier covers this easily (tens of thousands of
reads/writes per day, free forever, no card required). A chapter of ~50
people clicking buttons occasionally won't come close to the limit.

## Troubleshooting

- **Nothing happens when I log in / points don't save** — almost always
  means the Firestore rules (step 4 above) haven't been published yet, or
  `firebaseConfig` still has placeholder values. Open the browser console
  (F12) for the actual error.
- **My old test data disappeared** — if the login scheme changes (e.g.
  name-only → name+birthday), old records live under a different document
  key and won't show up under the new one. This only matters for data you
  care about keeping.
- **Two people, same name** — they'll only collide if they also share a
  birthday; otherwise each name+birthday pair gets its own record.
- **"No existing pearls found" popup** — this shows up whenever the name
  + birthday you typed doesn't match any existing account, so a typo
  doesn't silently create a duplicate empty account. If you're sure
  you've logged in before, check for typos (extra space, wrong birth
  year) and cancel instead of continuing.
- **Someone's totals look reset** — this version switched from one running
  number per category to a list of individual logged entries. Existing
  records are auto-migrated the first time they're loaded (their old
  total becomes one "Imported total" entry per category), so nothing is
  lost — but it will look different from before.
