# Teen Patti Pot

A shareable virtual chip tracker for Teen Patti game nights. No real money, no
physical chips — one link, everyone registers a name, bets and folds from
their own phone, and the host awards each round's pot. Chips and the pot stay
in sync live across every phone at the table.

This is a single static page (`index.html`) with no server of its own. It
uses a free [Firebase](https://firebase.google.com/) Firestore database as the
shared "table state," and **nobody needs to log in to anything** — not you,
not your friends. Firebase just needs to be connected once before the first
game.

## 1. Create a free Firebase project (~3 minutes)

1. Go to [console.firebase.google.com](https://console.firebase.google.com/)
   and sign in with any Google account.
2. Click **Add project**, give it any name (e.g. "teen-patti-pot"), and
   finish the wizard (you can decline Google Analytics, it's not needed).
3. In the left sidebar, open **Build → Firestore Database**, click
   **Create database**, and choose **Start in test mode** (you'll replace the
   rules with the ones below in a minute). Pick any region.
4. In the left sidebar, click the gear icon → **Project settings**. Under
   **Your apps**, click the **`</>`** (web) icon to register a new web app
   (any nickname is fine, you don't need Firebase Hosting).
5. Firebase will show a `firebaseConfig` object like this:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "teen-patti-pot-xxxxx.firebaseapp.com",
     projectId: "teen-patti-pot-xxxxx",
     storageBucket: "teen-patti-pot-xxxxx.appspot.com",
     messagingSenderId: "123456789012",
     appId: "1:123456789012:web:abcdef1234567890"
   };
   ```

## 2. Paste the config into `index.html`

Open `index.html`, find `FIREBASE_CONFIG` near the top of the `<script>`
block, and replace the placeholder values with the ones Firebase gave you:

```js
var FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "teen-patti-pot-xxxxx.firebaseapp.com",
  projectId: "teen-patti-pot-xxxxx",
  storageBucket: "teen-patti-pot-xxxxx.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abcdef1234567890"
};
```

This config is safe to commit and safe to be public — it's a client
identifier, not a secret. Access is controlled by the Firestore security
rules below, not by hiding this object.

## 3. Set the Firestore security rules

Back in the Firebase console, go to **Build → Firestore Database → Rules**
and replace the contents with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /games/{roomCode} {
      allow read, write: if true;
      match /{subcollection=**} {
        allow read, write: if true;
      }
    }
  }
}
```

Click **Publish**.

**What this means:** anyone who knows a table's room code (a random 5-character
code, not guessable in practice) can read and write that table's data — which
is exactly what lets your friends bet and fold without logging in. There's no
real money and no personal data beyond first names, so this is a reasonable
trade-off for a casual game night. Don't reuse this Firebase project for
anything sensitive.

## 4. Turn on GitHub Pages (if you haven't already)

If this repo was pushed by Claude Code, GitHub Pages is likely already
enabled and serving `index.html` from the repo root. If not: **Settings →
Pages → Source → Deploy from a branch → `main` / `/ (root)` → Save**. GitHub
will give you a `https://<username>.github.io/<repo>/` URL — that's the link
you share with friends.

## Playing

- Open the link, tap **Host a table**, pick a starting chip amount (1,000 /
  2,000 / 5,000 / custom), and create the table.
- Share the link (it carries your room code) or read out the code shown at
  the top of the table screen — friends tap **Join a table** and enter their
  name.
- Everyone bets and folds from their own phone. The host picks the winner
  each round and taps **Award pot**; chips update for everyone instantly.
- **Void round** refunds a round's bets if a hand gets misdealt.
- **End table for everyone** (host only) permanently deletes that table's
  data from Firestore.

## Local preview

Just open `index.html` in a browser — no build step, no dependencies to
install.
