# Tic‑Tac‑Toe — Firebase & Deployment Notes

This file contains quick guidance for Firestore security rules and deploying the static game to GitHub Pages.

## Firestore rules (development -> production)

During development you can use permissive rules for convenience, but lock them down for production.

Example (development - quick testing):

```rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // allow read/write to rooms collection while testing
    match /rooms/{roomId} {
      allow read, write: if true;
      match /{document=**} {
        allow read, write: if true;
      }
    }
  }
}
```

Example (safer production starting point): allow only limited writes from authenticated users and time-limited room creation.

```rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{roomId} {
      allow read: if true; // anyone can read room state to connect
      allow create: if request.time < timestamp.date(2099, 12, 31) && request.auth != null; // require auth to create (adjust as needed)
      allow update, delete: if request.auth != null;
      match /{document=**} {
        allow read, write: if request.auth != null;
      }
    }
  }
}
```

Notes:
- The correct policy depends on your desired UX (anonymous play vs. authenticated). If you need anonymous (no sign-in) quick-play, keep rules permissive but monitor usage and set quotas.
- For public games, consider adding server-side cleanup of old rooms or write lifecycle rules.

## Deploying to GitHub Pages (static site)

1. Create a repository on GitHub and push this project (or use this repo).
2. In GitHub, go to Settings → Pages.
3. Under "Source", choose the branch you want to publish (commonly `main`) and the folder (`/ (root)`) then Save.
4. GitHub will build and serve `index.html` at `https://<your-username>.github.io/<repo>/` (or a custom domain if configured).

Alternatively, you can use the `gh-pages` branch:

```bash
# from project root
git checkout -b gh-pages
git push origin gh-pages
# then set Pages source to gh-pages branch in repo settings
```

If you prefer an automated deploy, use GitHub Actions to build and push to `gh-pages` (not necessary for this plain static site).

## Troubleshooting Firebase init errors

- If the UI shows an error banner in the Auto signaling pane, check the message and ensure the `firebase-config` block in `index.html` contains your project's config.
- Make sure Firestore is enabled in the Firebase Console and that your project's projectId matches the config.
- If analytics fails to initialize (some browsers block it), the app will still try to initialize Firestore; analytics errors are handled gracefully.

## Security and costs

- Firebase API keys in client apps are normal and expected; they are not a secret by themselves. Protect your project by tightening Firestore rules and monitoring usage.
- If you expect public usage, set up billing/quotas and consider authenticated access or rate-limiting via Cloud Functions.

---
If you'd like, I can add a GitHub Actions workflow to auto-deploy to GitHub Pages when you push to `main`.
