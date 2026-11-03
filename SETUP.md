# Broken Squares: setup guide for IT

This folder holds a browser game for the Broken Squares team exercise. Players scan a QR code with their phone and play. They don't need an account or an app.

The game is one web page (`public/index.html`). Live play runs through a free Google Firebase project:

- **Anonymous sign-in** gives each phone a private ID without asking for anything.
- **Realtime Database** keeps every phone in sync.

Setup takes about 30 minutes and needs a Google account that can create Firebase projects.

## What's in the folder

| File | What it is |
|---|---|
| `public/index.html` | The whole game. You only edit one block near the top of its script (step 4). |
| `database.rules.json` | Security rules for the database (step 3). |
| `firebase.json` | Deployment settings for Firebase Hosting (step 5, Option A only). |

## 1. Create the Firebase project

1. Go to <https://console.firebase.google.com> and choose **Create a project**. Name it something like `denka-broken-squares`.
2. Google Analytics isn't needed. You can switch it off.
3. The free **Spark** plan is enough. See the capacity notes at the end.

## 2. Turn on anonymous sign-in

1. In the project, open **Build → Authentication → Get started**.
2. Under **Sign-in method**, choose **Anonymous**, switch it on and save.

## 3. Create the database and add the rules

1. Open **Build → Realtime Database → Create database**.
2. For location, choose **Singapore (asia-southeast1)** so data stays in-region.
3. Start in **locked mode**.
4. Open the **Rules** tab. Replace everything with the contents of `database.rules.json` and press **Publish**.

The rules allow only signed-in devices, which every player is after the silent sign-in. A player can only take an empty seat. Only someone with a session's facilitator PIN can start, pause, end, reset or delete that session. The PIN itself can never be read back.

## 4. Connect the page to your project

1. Open **Project settings** (gear icon) → **General** → **Your apps** and add a **Web app** (the `</>` icon). Firebase Hosting can be ticked here or set up in step 5.
2. Firebase shows a `firebaseConfig` block. Open `public/index.html` in a text editor and find this block near the top of the main script:

   ```js
   const FIREBASE_CONFIG={
     apiKey: "",
     authDomain: "",
     databaseURL: "",
     projectId: "",
     appId: ""
   };
   ```

3. Copy the five values across. **`databaseURL` must be filled in.** For a Singapore database it looks like `https://denka-broken-squares-default-rtdb.asia-southeast1.firebasedatabase.app`. If Firebase didn't show it, copy it from the top of the Realtime Database **Data** tab.

These values are meant to be public. They identify the project but grant no access, because the database rules do the protecting.

## 5. Put the page on the web

Phones need to reach the page over **HTTPS**. Pick one option.

**Option A: Firebase Hosting (simplest).** From this folder, on a computer with Node.js:

```bash
npm install -g firebase-tools
firebase login
firebase use --add            # pick the project from step 1
firebase deploy               # publishes public/ and the database rules
```

The game is then at `https://<project-id>.web.app`.

**Option B: your own web server or intranet.** Upload `public/index.html` to any HTTPS location that players' phones can reach. Then, in Firebase, go to **Authentication → Settings → Authorized domains** and add that domain.

A page on an internal-only network won't work for phones on mobile data. Either check that players can join the same Wi-Fi, or use Option A.

**Option C: GitHub Pages (free, no command line).** GitHub Pages only serves the page, so you still need steps 1–4 for the database. Skip `firebase deploy`.

1. On <https://github.com>, create a new **public** repository, for example `broken-squares`. Free GitHub Pages needs a public repo. That's fine: the Firebase values in the page are meant to be public, and the database rules do the protecting.
2. Choose **Add file → Upload files**. Upload `public/index.html`, keep the name `index.html`, put it at the top level of the repo, and commit.
3. Open **Settings → Pages**. Under **Build and deployment**, choose **Deploy from a branch**, then **main** and **/ (root)**, and save.
4. After a minute or two the game is live at `https://<your-username>.github.io/broken-squares/`. The address appears at the top of the Pages settings.
5. In Firebase, open **Authentication → Settings → Authorized domains → Add domain** and add `<your-username>.github.io`. Leave out the `https://` and the `/broken-squares/` part. Without this step, phones show "Couldn't connect to the game."

To change the game later, upload a new `index.html` to the same repo. It goes live within a couple of minutes, and sessions already running carry on. If your organisation has a GitHub organisation account, create the repo there instead of under a personal account. The steps are the same, and the address becomes `https://<org-name>.github.io/broken-squares/`.

The QR codes always use whatever address the page is opened at, so nothing in the page needs changing for GitHub Pages.

## 6. Check it works (5 minutes)

1. On a laptop, open the game, choose **Run a session**, set 1 team and a PIN, and press **Create session**.
2. Scan the QR code with a phone. The seat picker for that session should open. Type a name and take seat 1.
3. Scan with a second phone and take seat 2. Seat 1 should show as taken.
4. On the laptop, press **Give the start signal**. Both phones count down from 3 together and unlock.
5. On phone 1, tap a piece, then tap **2** under "Give to seat". The piece should appear on phone 2 within a second.
6. On the laptop, choose **Delete session** after ending the round.

If a phone says it can't connect, check that anonymous sign-in is on (step 2) and that the domain is authorized (step 5, Option B or C).

## Capacity and cost

- **Free Spark plan:** up to **100 phones and screens connected at once**. That's about 18 teams of five, plus the facilitator and a few observers. Data use for a workshop is tiny.
- **Bigger events:** switch the project to the **Blaze** (pay-as-you-go) plan. That lifts the limit to 200,000 connections, and a workshop typically costs nothing to a few cents.

## Data and privacy

The database stores only what the game needs: the session name, the first names or nicknames players type, a random device ID, piece positions, the times of gives, and observer notes. No emails, phone numbers or Google accounts are collected.

- Ask players to use a first name or nickname.
- Delete each session after the debrief. **Delete session** on the facilitator screen removes everything for that session.
- The database location chosen in step 3 decides where data is stored.

## Updating the game later

Edit `public/index.html` and run `firebase deploy` again, or re-upload the file. Sessions already in the database carry on working.
