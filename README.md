# Teen Patti Pot

A shareable virtual chip tracker for Teen Patti game nights. No real money, no
physical chips — one link, everyone registers a name and avatar, bets and
folds from their own phone on their own turn, and the host runs each round
(boot, deal, award). Chips, cards, and the pot stay in sync live across every
phone at the table.

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
5. Firebase will show a `firebaseConfig` object — copy the whole thing.

## 2. Paste the config into `index.html`

Open `index.html`, find `FIREBASE_CONFIG` near the top of the `<script>`
block, and replace the placeholder values with the ones Firebase gave you.
**Keep the variable name exactly `FIREBASE_CONFIG`** (all caps) — Firebase's
own snippet calls it `firebaseConfig`, but the rest of this file's code reads
`FIREBASE_CONFIG`, so renaming it is required, not optional:

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
    match /completed_games/{roomCode} {
      allow read, write: if true;
    }
    match /profiles/{deviceId} {
      allow read, write: if true;
    }
  }
}
```

Click **Publish**.

**What this means:** anyone who knows a table's room code (a random
6-character code, not guessable in practice — optionally add a table PIN
too, see below) can read and write that table's data — which is exactly what
lets your friends bet and fold without logging in. `completed_games` holds
the read-only archive created when a host ends a table; `profiles` holds each
device's lifetime stats (games played, net chips), keyed by a random ID
generated in that browser's local storage, with no name-to-person link
beyond whatever name someone typed in. There's no real money and no personal
data beyond first names, so this is a reasonable trade-off for a casual game
night. Don't reuse this Firebase project for anything sensitive.

## 4. Turn on GitHub Pages (if you haven't already)

If this repo was pushed by Claude Code, GitHub Pages is likely already
enabled and serving `index.html` from the repo root. If not: **Settings →
Pages → Source → Deploy from a branch → `main` / `/ (root)` → Save**. GitHub
will give you a `https://<username>.github.io/<repo>/` URL — that's the link
you share with friends.

## Playing

- Open the link, tap **Host a table**, pick an avatar, a starting chip
  amount, a boot/ante (or 0 to skip it), and an optional PIN, then create the
  table.
- Share the link (it carries your room code) or tap the 📱 icon for a QR
  code your friends can scan — they tap **Join a table**, enter their name
  and avatar, and (if you set one) the PIN.
- **Start round** (host only) collects the boot from every player and,
  if you tick the box, deals 3 hidden cards to each player. Players can tap
  **View my cards** to reveal their own hand and see its rank (Trail, Pure
  Sequence, Sequence, Color, Pair, or High Card) — nobody else sees your
  cards, just a face-down back and a Blind/Seen badge.
- Betting is turn-based — only the highlighted player can bet or fold, and
  the turn passes automatically afterward. Blind bets match the current
  stake; betting after viewing your cards ("Seen") suggests double — both are
  just suggested quick-bet amounts, you can still type any amount.
- The host picks the winner and taps **Award pot**; chips, the leaderboard,
  and round history update for everyone instantly. **Void round** refunds a
  misdealt round instead.
- **Rebuy** appears for anyone at 0 chips, topping them back up to the buy-in.
- **Export CSV** downloads the final standings and round history.
- **End table for everyone** (host only) archives the table to Past Games
  (visible next time that device opens the app) and permanently removes the
  live table from Firestore.
- The 🔊/🌙 icons in the header toggle sound/haptics and light/dark theme —
  both remembered on that device. A lifetime stats line ("🏆 Lifetime: N
  games...") appears on the home screen once you've played at least one game
  on that device.

## Local preview

Just open `index.html` in a browser — no build step, no dependencies to
install.
