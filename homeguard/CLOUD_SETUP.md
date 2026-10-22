# Cloud dashboard: Firebase + GitHub Pages

Result: a link like `https://YOURNAME.github.io/homeguard/` that works from any phone or PC,
with login, live status, and a searchable event history with snapshot thumbnails.
No port-forwarding. The Pi only makes outgoing connections.

```
Pi (detects) ──► Firebase Firestore ◄── GitHub Pages website (you log in)
```

What stays on the Pi: live video (too heavy for Firebase's free plan) and the local site at
`http://<pi-ip>:5000`. What goes to the cloud: events, small thumbnails, and an "is the Pi online" status.
Email alerts still work exactly as before.

## Part 1: Firebase (5-10 min)

1. Go to https://console.firebase.google.com -> **Add project** (turn Google Analytics off).
2. **Build -> Firestore Database -> Create database**. Choose *production mode* and a region near you.
3. **Build -> Authentication -> Get started -> Sign-in method -> Email/Password -> Enable**.
4. **Authentication -> Users -> Add user**: create a login for each homeowner (email + password).
   There is no public sign-up page, so only users you add here can log in.
5. **Firestore -> Rules**: paste the contents of `firestore.rules`, replace `homeowner@example.com`
   with the same email(s) from step 4, and press **Publish**.
6. **Project settings (gear) -> Service accounts -> Generate new private key**.
   Save the downloaded file as `firebase-key.json` inside your `homeguard` folder on the Pi.
   (This key is a SECRET: never upload it to GitHub.)
7. **Project settings -> General -> Your apps -> Web (`</>`)** -> register an app (skip Hosting).
   Copy the `firebaseConfig` values into `docs/firebase-config.js`.

## Part 2: Connect the Pi

```bash
cd ~/homeguard
./venv/bin/pip install firebase-admin      # skip if install.sh already did it
nano .env                                  # add the line:  FIREBASE_CRED=firebase-key.json
./venv/bin/python app.py
```
On start you should see `[cloud] Firebase sync ON`. Use the dashboard's
"Simulate PIR motion" button, or walk in front of the camera, and watch Firestore -> Data
fill with an `events` collection and a `status/current` document.

## Part 3: GitHub Pages

1. Create a new **public** repository on GitHub (e.g. `homeguard`). Pages is free for public repos.
2. Upload the **contents of the `docs/` folder** (`index.html` and `firebase-config.js`) to the repo root.
   Do NOT upload `.env` or `firebase-key.json`. The included `.gitignore` protects you if you push the whole project.
3. Repo **Settings -> Pages -> Build and deployment -> Deploy from a branch -> `main` / `(root)` -> Save**.
4. After about a minute your link appears there: `https://YOURNAME.github.io/homeguard/`.
5. If login ever complains about the domain: Firebase -> Authentication -> Settings ->
   **Authorized domains** -> add `YOURNAME.github.io`.

## "Searchable"

- **Inside the site**: the search box filters your events by text, level, or date.
- **On Google**: the sign-in page is public, so it can be found once Google crawls it
  (submit the link in Google Search Console to speed this up). Your **events and photos are not
  public**: they sit behind login and `firestore.rules`.
- Never put live camera footage or snapshots on a public page.

## Free-plan limits (Firestore Spark: about 20k writes / 50k reads per day)

HomeGuard writes one status update per minute (about 1,440/day) plus one write per logged event
(Level 1/2 at most every 30 s), which fits comfortably. Old events are not auto-deleted; clear them
in the Firebase console now and then if you want to keep storage small.

## Security checklist
- Strong passwords on the Firebase users; only list real homeowner emails in `firestore.rules`.
- Keep `firebase-key.json` and `.env` off GitHub.
- Tell household members and guests that cameras are in use.
