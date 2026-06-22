# FINTECH — Personal Finance Dashboard

> Personal. Finance. Reimagined.

A single-page, multi-user personal finance ledger built on Firebase. Manual-entry by design — no bank-linking, no Plaid, no third party holding your credentials. Each signed-in user gets a fully isolated dataset, enforced at the database level, not just hidden in the UI.

**Live:** [joshstafki.github.io/fintech](https://joshstafki.github.io/fintech/)

---

## Features

- **Real-time sync** — built on Firestore's `onSnapshot` listeners, not polling. Edits reflect instantly across every open tab/device.
- **Multi-user via Google Sign-In** — anyone can sign in with their Google account; each user's data lives in its own Firestore subtree and is invisible to everyone else.
- **Session persistence** — sign in once, stay signed in across browser restarts (handled automatically by Firebase Auth).
- **Inline-editable ledger** across five tracked categories:
  | Category | Purpose |
  |---|---|
  | `Out_Of_Pocket` | Cash/personal-card expenses |
  | `Bank_Initiated` | Bank-side transactions |
  | `Fill_Ups` | Fuel/gas tracking |
  | `Bank_Reconciliation` | Running bank balance checks |
  | `Grocery_Budget` | Grocery spend vs. budget |
- **Configurable budget settings** — spendable income, starting balances, editable inline.
- **Revenue/expense chart** via Chart.js.
- **Hard reset** — a one-click wipe of a user's own data (does not touch anyone else's).
- **Branded login screen** — animated gradient, glassmorphic card, and logo lockup matching the FINTECH visual identity.

---

## Tech Stack

- Vanilla HTML/CSS/JS — no build step, no framework, no bundler
- **Firebase Authentication** (Google provider)
- **Cloud Firestore** for data storage
- **Chart.js** for visualizations
- **Space Grotesk** / **JetBrains Mono** (Google Fonts) for the login screen's type

---

## Data Model

```
users/{uid}/
  entries/{entryId}        → { category, name, amount, date, paid, ... }
  settings/
    budget                 → { spendableIncome, bankReconStart, groceryStart }
```

Every document a user can touch lives under their own `uid`. There is no shared top-level collection — isolation is structural, not a query filter.

---

## Security Rules

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null
                          && request.auth.uid == userId
                          && request.auth.token.email_verified == true;
    }
  }
}
```

This is the actual enforcement layer — a user can only ever read or write documents under `users/{their-own-uid}`, regardless of what the client-side code tries to do.

---

## Setup

1. **Create a Firebase project** at [console.firebase.google.com](https://console.firebase.google.com).
2. **Enable Google as a sign-in provider** — Authentication → Sign-in method → Google.
3. **Add your hosting domain** to Authentication → Settings → Authorized domains (required for the sign-in popup to work).
4. **Publish the Firestore rules** above under Firestore → Rules.
5. **Drop in your Firebase config** — replace the `firebaseConfig` object in `ledger_review.html` with your own project's values (these are public-facing by design; they identify your project, they don't grant access — Firestore rules do that).
6. **Serve the file** — it's static, so GitHub Pages, Firebase Hosting, or any static host works. No server-side code required.

---

## Roadmap

- [ ] iOS wrapper via Capacitor for App Store distribution
- [ ] CSV/spreadsheet export
- [ ] Optional Plaid-based bank linking (bigger compliance lift — would need real privacy policy/ToS work first)
- [ ] Onboarding flow explaining the five categories to new users

---

## Author

Built by Josh Stafki.
