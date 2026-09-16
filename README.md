# budget-ledger

# Shared household setup

About fifteen minutes, all in a browser. Until you finish step 8 the ledger keeps
working exactly as it does now, saving only on the device you're using. Nothing
breaks while you're partway through.

---

## 1. Create the Firebase project

1. Go to <https://console.firebase.google.com> and sign in with your Google account.
2. **Add project** → name it anything (`household-ledger` works).
3. Turn **Google Analytics off**. You don't need it and it only adds prompts.

## 2. Register a web app

1. On the project overview, click the **`</>`** (web) icon.
2. Nickname it `ledger`. **Do not** tick "Also set up Firebase Hosting" — GitHub
   Pages is already hosting this.
3. Firebase shows you a `firebaseConfig` block. Leave this tab open; you need it
   in step 8.

## 3. Turn on email sign-in

1. Left sidebar → **Build** → **Authentication** → **Get started**.
2. **Sign-in method** tab → **Email/Password** → enable the top toggle → **Save**.
   Leave "Email link (passwordless)" off.

> Why not "Sign in with Google"? It needs a popup or a redirect, and both are
> unreliable inside an iOS home-screen web app. Email and password just works
> there.

## 4. Create your two accounts

1. **Users** tab → **Add user**. Enter your email and a password. Repeat for your wife.
2. You're inventing these passwords — they're for this app only, not your Google
   passwords. Put them in your password manager.
3. Each row now shows a **User UID**, a long string like `k3Jd9...`. Copy both.
   You need them in the next step.

## 5. Create the database

1. **Build** → **Firestore Database** → **Create database**.
2. Choose a location near you. This can't be changed later, but nothing here is
   latency-sensitive.
3. Pick **Start in production mode** — locked down by default, which is what you want.

## 6. Lock it to just the two of you

**Rules** tab → replace everything with this, pasting your two UIDs from step 4:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /budgets/household {
      allow read, write: if request.auth != null
        && request.auth.uid in ['PASTE_YOUR_UID', 'PASTE_HER_UID'];
    }
  }
}
```

Click **Publish**. Now only those two accounts can read or write the budget —
not the public, not anyone else who somehow got an account on the project.

## 7. Authorize your domain

**Authentication** → **Settings** → **Authorized domains** → **Add domain** →
`YOUR-USERNAME.github.io`.

## 8. Paste the config

Open `index.html`. The block you want is at the very top of the scripts, about
two-thirds of the way down the file:

```js
window.FIREBASE_CONFIG = {
  apiKey: "",
  authDomain: "",
  projectId: "",
  appId: ""
};
```

Fill in those four values from step 2. Ignore any other keys Firebase gave you —
`storageBucket`, `messagingSenderId`, `measurementId` aren't needed.

Commit and push.

**These keys are safe in a public repo.** A Firebase web config isn't a secret;
it only identifies the project. Your step 6 rules are what keep the data private.

## 9. Both of you install it

On each phone: Safari → `https://YOUR-USERNAME.github.io/budget-ledger/` →
Share → **Add to Home Screen**. Open it, sign in once. It stays signed in.

If you already have the old icon on your home screen, delete it first — it's
pinned to the old copy and will keep showing you that.

---

## How it behaves day to day

- **Changes appear on the other phone in a second or two.** No refresh needed.
- **If you're both editing at once, the last save wins.** The app won't overwrite
  a field while someone's actually typing in it — it waits until you tap away —
  but it can't merge two people editing the same number. In practice you'll
  rarely collide.
- **It works offline.** Type on the subway; it syncs when you're back.
- **The Data tab tells you the truth.** It says "Synced" when the cloud has your
  numbers. If it says anything else, believe it.

## Things worth knowing

- **Still back up.** Data tab → *Back up to a file*, every month or so. Cloud
  sync protects against losing a phone, not against one of you clearing
  everything by mistake. Keep the backup out of the repo — it has real balances
  in it, and the repo is public.
- **Free tier is far more than enough.** This writes one small document. You will
  not approach the limits.
- **Adding her income:** Income tab → *Add another income*. Set her pay cadence
  and next pay date separately from yours; the plan folds both into each period.

## If sign-in fails

| What you see | What it means |
| --- | --- |
| "No account for that email" | Not added in step 4, or a typo |
| "Wrong email or password" | Reset it under Authentication → Users → ⋮ |
| "Sync error: permission-denied" | A UID in the step 6 rules is wrong or missing |
| Stuck on "Connecting to your household…" | `projectId` is wrong, or you're offline |

Anything else shows up on the **Data** tab under "Problems this session" with a
timestamp. Send me what's there.
