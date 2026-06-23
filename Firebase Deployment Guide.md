# 🔥 Firebase Deployment Guide

A complete step-by-step guide to deploy a project on **Firebase** (Hosting, Functions, Firestore, Storage).
Keep this as a personal reference note in your GitHub repo.

---

## 📋 Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Install Firebase CLI](#2-install-firebase-cli)
3. [Login to Firebase](#3-login-to-firebase)
4. [Create / Select a Firebase Project](#4-create--select-a-firebase-project)
5. [Initialize Firebase in Your Project](#5-initialize-firebase-in-your-project)
6. [Build Your Project](#6-build-your-project)
7. [Deploy to Firebase Hosting](#7-deploy-to-firebase-hosting)
8. [Deploy Firestore Rules & Indexes](#8-deploy-firestore-rules--indexes)
9. [Deploy Cloud Functions](#9-deploy-cloud-functions)
10. [Deploy Storage Rules](#10-deploy-storage-rules)
11. [Deploy Everything at Once](#11-deploy-everything-at-once)
12. [Useful Firebase CLI Commands](#12-useful-firebase-cli-commands)
13. [Common Errors & Fixes](#13-common-errors--fixes)
14. [Quick Checklist](#14-quick-checklist)

---

## 1. Prerequisites

- A [Firebase account](https://console.firebase.google.com/) (use your Google account)
- [Node.js](https://nodejs.org/) installed (LTS version recommended)
- Your project code ready (React, Vue, Angular, plain HTML/CSS/JS, etc.)
- npm or yarn installed

Check versions:

```bash
node -v
npm -v
```

---

## 2. Install Firebase CLI

Install the Firebase Command Line Tools globally:

```bash
npm install -g firebase-tools
```

Verify installation:

```bash
firebase --version
```

---

## 3. Login to Firebase

Login with your Google account from the terminal:

```bash
firebase login
```

This opens a browser window — sign in and grant access.

> If you're on a server/VM without a browser, use:
> ```bash
> firebase login --no-localhost
> ```

---

## 4. Create / Select a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **Add Project** → give it a name → follow the setup steps
3. Once created, note your **Project ID**

To list projects from CLI:

```bash
firebase projects:list
```

---

## 5. Initialize Firebase in Your Project

Navigate to your project folder:

```bash
cd my-project
```

Run the init command:

```bash
firebase init
```

You'll be asked which features to set up. Select using **arrow keys + spacebar**:

```
◉ Hosting: Configure files for Firebase Hosting
◉ Firestore: Deploy rules and create indexes
◉ Functions: Configure a Cloud Functions directory
◉ Storage: Deploy Cloud Storage security rules
```

Then follow prompts:

| Prompt | What to choose |
|---|---|
| Select a project | Choose existing project or create new |
| Public directory | `build`, `dist`, or `public` (depends on framework) |
| Single-page app? | `Yes` if using React/Vue/Angular router |
| Set up automatic builds with GitHub? | `No` (unless you want CI/CD) |
| Overwrite index.html? | `No` (if it already exists) |

This creates:
- `firebase.json`
- `.firebaserc`
- `firestore.rules` (if selected)
- `storage.rules` (if selected)

---

## 6. Build Your Project

If you're using a frontend framework, build the production version first.

**React:**
```bash
npm run build
```

**Vue:**
```bash
npm run build
```

**Angular:**
```bash
ng build --configuration production
```

Make sure the `firebase.json` `"public"` field points to this build folder (e.g. `build` or `dist`).

---

## 7. Deploy to Firebase Hosting

Deploy your site:

```bash
firebase deploy --only hosting
```

After deployment, you'll get a live URL like:

```
https://your-project-id.web.app
```

---

## 8. Deploy Firestore Rules & Indexes

If your project uses Firestore:

```bash
firebase deploy --only firestore:rules
firebase deploy --only firestore:indexes
```

Or both together:

```bash
firebase deploy --only firestore
```

---

## 9. Deploy Cloud Functions

Navigate to the functions folder and install dependencies:

```bash
cd functions
npm install
cd ..
```

Deploy all functions:

```bash
firebase deploy --only functions
```

Deploy a specific function:

```bash
firebase deploy --only functions:functionName
```

---

## 10. Deploy Storage Rules

```bash
firebase deploy --only storage
```

---

## 11. Deploy Everything at Once

```bash
firebase deploy
```

This deploys Hosting + Firestore + Functions + Storage (whatever is configured in `firebase.json`).

---

## 12. Useful Firebase CLI Commands

| Command | Description |
|---|---|
| `firebase login` | Login to Firebase |
| `firebase logout` | Logout |
| `firebase projects:list` | List all projects |
| `firebase use <project-id>` | Switch active project |
| `firebase init` | Initialize Firebase features |
| `firebase serve` | Run local server (Hosting + Functions emulator) |
| `firebase emulators:start` | Start local emulator suite |
| `firebase deploy` | Deploy everything |
| `firebase deploy --only hosting` | Deploy only hosting |
| `firebase hosting:channel:deploy preview` | Deploy a preview channel (staging link) |
| `firebase functions:log` | View Cloud Functions logs |

---

## 13. Common Errors & Fixes

| Error | Fix |
|---|---|
| `Error: HTTP Error: 403` | Run `firebase login --reauth` |
| `Error: No currently active project` | Run `firebase use --add` and select project |
| `command not found: firebase` | Re-run `npm install -g firebase-tools`, restart terminal |
| Blank page after deploy (SPA) | Ensure `"rewrites"` is set in `firebase.json` to redirect all routes to `index.html` |
| Functions deploy fails (Node version) | Set `"engines": { "node": "18" }` in `functions/package.json` |
| Old build deployed | Re-run `npm run build` before `firebase deploy` |

**Example `firebase.json` rewrite rule for SPA:**

```json
{
  "hosting": {
    "public": "build",
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```

---

## 14. Quick Checklist

- [ ] Firebase CLI installed (`firebase --version`)
- [ ] Logged in (`firebase login`)
- [ ] Project created in Firebase Console
- [ ] `firebase init` run, correct features selected
- [ ] Production build created (`npm run build`)
- [ ] `firebase.json` points to correct build folder
- [ ] Deployed (`firebase deploy`)
- [ ] Live URL tested in browser

---

### 📌 Notes
- Free tier (Spark plan) supports Hosting + Firestore + limited Functions usage.
- Cloud Functions deployment requires the **Blaze (pay-as-you-go) plan**.
- Use `firebase hosting:channel:deploy` for preview links before going live on production.

---

*Last updated: keep this file versioned in your repo and update commands as Firebase CLI evolves.*
