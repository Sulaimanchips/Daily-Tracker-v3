# Daily Tracker — account + sync setup

One-time, about 8 minutes. After this you'll have a real email/password
account, and your checklist, streaks, sleep scores and steps will follow you
to every device you sign in on.

**Before you start:** use a personal Gmail account, not a managed Workspace
one. Firebase projects live inside Google Cloud, and managed accounts are
blocked from creating them — same wall you hit with the Calendar API.

## 1. Create the project
Go to https://console.firebase.google.com → **Add project** → name it
(e.g. `daily-tracker`) → skip Google Analytics → Create.

## 2. Turn on email sign-in
Left sidebar: **Build → Authentication → Get started** →
**Sign-in method** tab → **Email/Password** → toggle Enable → Save.

Leave "Email link (passwordless sign-in)" off.

## 3. Create the database
Left sidebar: **Build → Firestore Database → Create database** →
**Start in production mode** → pick the location closest to you
(`europe-west1` or `me-central2` if offered) → Enable.

Production mode denies everything by default. Step 4 opens it up to you only.

## 4. Lock the data to each account
Firestore → **Rules** tab → replace everything with:

    rules_version = '2';
    service cloud.firestore {
      match /databases/{database}/documents {
        match /users/{uid} {
          allow read, write: if request.auth != null && request.auth.uid == uid;
        }
      }
    }

**Publish.** This is the line that makes it actually private: a signed-in
account can only touch its own document, and nobody signed out can touch
anything.

## 5. Copy your web config
Gear icon (top left) → **Project settings** → scroll to **Your apps** →
click the **`</>`** (web) icon → give it any nickname → **do not** tick
Firebase Hosting → Register app.

You'll get a `firebaseConfig` block with six values.

## 6. Paste it into index.html
Open `index.html`, find this near the top of the `<script>` section:

    const FIREBASE_CONFIG = {
      apiKey: "PASTE_API_KEY",
      ...
    };

Replace each `PASTE_...` with the matching value. Save.

The apiKey is not a secret — it identifies your project, it doesn't grant
access. The rules in step 4 are what protect your data.

## 7. Authorise your domain
Back in **Authentication → Settings → Authorized domains**, add the domain
you deploy to (e.g. `yourname.github.io`). `localhost` is already there.

Skip this and sign-in will fail with an unauthorised-domain error.

## 8. Deploy and create your account
Upload all the files (`index.html`, `manifest.json`, `sw.js`, both icons).
Open the site, tap **Create an account**, enter your first name, email and a
password of at least 6 characters.

Then open the same site on your laptop, sign in with the same details, and
tick something — it should appear on your phone within a second or two.

## Notes

- **Staying signed in.** The session persists, so you sign in once per device
  and the app opens straight to your day after that.
- **Signing out** wipes this device's cached copy of your data. Your cloud
  copy is untouched — sign back in and it all returns.
- **Two accounts, one phone** each get a separate slice of local storage, so
  nothing bleeds across.
- **Anything you tracked before** this update gets pulled into the first
  account you sign in with, then pushed to the cloud.
- **Offline** still works. Ticks save locally and sync when you're back on.
  There's an "Use offline on this device only" link if you ever want to skip
  signing in — that mode has its own separate, unsynced data.
- **After re-uploading files**, close and reopen the app twice — the service
  worker serves the cached copy first and updates in the background.
