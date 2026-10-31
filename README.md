# Trip Kitty — Android and iPhone app

Trip Kitty helps friends split trip spendings. This folder turns it into a real phone app with **Capacitor** (a free tool that puts a web app inside an Android or iPhone app).

## What is in this folder

| File / folder | What it is |
|---|---|
| `www/index.html` | The whole app (screens, maths, saving) |
| `www/firebase-config.js` | Online sync settings. Empty = no sync |
| `assets/` | App icon and splash screen pictures |
| `capacitor.config.json` | App name and app ID |
| `firestore.rules` | Security rules for online sync |
| `.github/workflows/android.yml` | Builds the Android app for you on GitHub |

## Two ways the app can save data

1. **On this phone only (default).** Works with no internet and no setup. Each phone keeps its own trips. Friends can't see your trip, but you can use **Share summary** to send the result to your group chat.
2. **Synced online (optional).** Every trip gets a **trip code**. Friends install the app, tap **Trips → Join a friend's trip**, and paste the code. Then everyone sees and adds spendings to the same trip. This needs a free Firebase account (steps below).

---

## Step 1 — Change the app ID (do this first)

Open `capacitor.config.json` and change `"com.tripkitty.app"` to something only you own, for example `"in.yourname.tripkitty"`. The stores do not allow two apps with the same ID, and you can't change it after you publish.

## Step 2 — Get a test app on your Android phone (easiest way, no computer setup)

1. Make a free account on **github.com**.
2. Create a new repository and upload everything in this folder (keep the `.github` folder — it is hidden on some computers).
3. Open the **Actions** tab. The build starts by itself and takes about 5–10 minutes.
4. When it shows a green tick, open the build and download **trip-kitty-test-apk**.
5. Unzip it, send the `.apk` file to your Android phone, and open it. Android will ask you to allow "install unknown apps" — allow it for this one file.

## Step 3 — Publish on Google Play

1. Create a Google Play developer account (one-time fee of $25).
2. Make a signing key on any computer with Java:
   `keytool -genkey -v -keystore release.keystore -alias tripkitty -keyalg RSA -keysize 2048 -validity 10000`
   **Keep this file and its password safe forever.** You need it for every update.
3. Turn it into text: `base64 -w0 release.keystore` (on Mac: `base64 -i release.keystore`).
4. In GitHub: **Settings → Secrets and variables → Actions**, add 4 secrets:
   `KEYSTORE_BASE64` (the text from step 3), `KEYSTORE_PASSWORD`, `KEY_ALIAS` (`tripkitty`), `KEY_PASSWORD`.
5. Run the build again. Now you also get **trip-kitty-play-store-aab**. Upload the `.aab` file in the Google Play Console.

Google will also ask for: screenshots, a short description, a privacy policy link, and the "Data safety" form. If you turn on online sync, say that the app stores names, emails or phone numbers that users type in.

## Step 4 — iPhone (needs a Mac)

Apple only lets you build iPhone apps on a Mac with **Xcode** (free from the Mac App Store). You also need an Apple Developer account ($99 per year).

On the Mac, in this folder:

```
npm install
npx cap add ios
npm run icons -- --ios
npm run ios
```

Xcode opens. Choose your team under **Signing & Capabilities**, connect your iPhone, and press ▶. To publish, use **Product → Archive** and upload to App Store Connect.

## Working on your own computer (Android)

Install **Node.js** and **Android Studio**, then:

```
npm install
npx cap add android
npm run icons -- --android
npm run android
```

Android Studio opens. Press ▶ to run on a phone or emulator. After you change `www/index.html`, run `npx cap sync` again.

---

## Turn on online sync (optional)

1. Go to **console.firebase.google.com** and create a project (free "Spark" plan is enough for many trips).
2. **Build → Firestore Database → Create database.** Choose a location near your users (for India: `asia-south1`).
3. Open the **Rules** tab, delete what is there, paste everything from `firestore.rules`, and press **Publish**.
4. **Project settings → Your apps → Add app → Web (</>)**. Copy the settings it shows.
5. Paste them into `www/firebase-config.js`, like this:

```js
window.TRIP_KITTY_FIREBASE = {
  apiKey: "AIza...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  appId: "1:123:web:abc",
};
```

6. Upload the changed file to GitHub (or run `npx cap sync`) and build again.

The top of the app will now say **Synced online**, and each trip shows its code on the **Friends** screen.

**About safety:** the trip code is long and random, and the rules stop anyone from listing all trips, so only people with the code can open a trip. But anyone who has the code can change or delete it — like a shared Google Doc link. For a bigger public app, add Firebase sign-in later so each trip has an owner.

## Good to know

- The app needs internet the first time it starts with sync on. After that it also works offline and catches up later.
- The fonts load from Google Fonts. With no internet the app uses the phone's normal font. Everything still works.
- On Android, the back button goes to the trip list first, then closes the app.
