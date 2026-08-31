# AEPhi CO26 Pearl Tracker

A single-page web app for tracking Fall 2026 pearl points per CO26 member.
No password — sisters log in with their full name and birthday, which
together identify their counter. Data lives in a free Firebase Firestore
database so it's persistent and shared across everyone's phones/laptops.
The browser remembers the login (via localStorage), so returning to the
page on the same device logs you back in automatically — "Switch User"
clears it and returns to the login screen.

**Nothing about this is secure or authenticated.** Full name + birthday is
just enough to avoid two people accidentally sharing one counter and to
not be trivially guessable — anyone who actually knows both can still edit
that person's counter. That's intentional per the request — this is for
casual internal tracking, not anything official or sensitive.

## What's in here

- `index.html` — the entire app (no build step, no server). Open it,
  or host it on GitHub Pages.
- Categories and point values are pulled directly from the Fall 2026
  Points Breakdown PDF for **MC26** (10 categories, 30 pearls total).

## One-time setup (about 5 minutes)

You need a free Firestore database for the data to live in. Do this once:

1. Go to **console.firebase.google.com** and sign in with any Google account.
2. Click **Add project** → give it any name (e.g. `aephi-pearls`) →
   you can decline Google Analytics if asked → **Create project**.
3. In the left sidebar, click **Build → Firestore Database** →
   **Create database**.
   - Choose any location (closest to you is fine).
   - Start in **production mode** (we'll open it up manually next, so it
     never expires — this avoids the 30-day auto-lock that "test mode"
     has).
4. Once created, go to the **Rules** tab of Firestore and replace the
   contents with:
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
6. It'll show a `firebaseConfig` object like:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "aephi-pearls.firebaseapp.com",
     projectId: "aephi-pearls",
     storageBucket: "aephi-pearls.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
   Copy that whole block.
7. Open `index.html` in this repo, find the section near the top of the
   `<script type="module">` that says:
   ```js
   // ---- PASTE YOUR FIREBASE CONFIG HERE (see README.md) ----
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     ...
   };
   ```
   and replace it with the block you copied.

This config is **not a secret** — it's meant to be public in client-side
code. Access control comes from the Rules step above, not from hiding
this object.

## Deploying with GitHub Pages

1. Commit and push `index.html` (with your config pasted in) to this repo.
2. In the GitHub repo, go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to `Deploy from a branch`,
   pick the branch this file is on, and folder `/ (root)`.
   Save.
4. GitHub gives you a URL like `https://yourusername.github.io/PointTrackerAEPhi/`
   after a minute or two. Share that link with the chapter.

## How it works

- Each name typed (case-insensitive, whitespace-trimmed) maps to one
  document in Firestore (`counters/<name>`).
- Category buttons match the exact point values from the Fall 2026
  breakdown (e.g. Sisterhood Event = +2, Study Hours = +1, Lead a 2nd Ave
  = +2, etc).
- Each category has its own progress bar (current / target), plus an
  overall "X / 30 pearls" bar at the top.
- A "Miscellaneous" section covers non-category points (announcing,
  tabling, banner painting, etc.) that count toward the 30 total but
  don't belong to a specific pearl category.
- A `-1` button on each category is there for correcting mistakes.
- Updates sync live — if a sister edits her pearls on her phone, anyone
  else with her name loaded sees the change without refreshing.

## Costs

Firebase's free "Spark" tier covers this easily (tens of thousands of
reads/writes per day, free forever, no card required). A chapter of ~50
people clicking buttons occasionally won't come close to the limit.
