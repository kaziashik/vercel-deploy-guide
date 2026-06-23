# 🔥 Firebase Deployment Guide

A complete step-by-step guide to deploy a project on **Firebase** (Hosting, Functions, Firestore, Storage).

---

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Install Firebase CLI](#install-firebase-cli)
3. [Login to Firebase](#login-to-firebase)
4. [Create / Select a Firebase Project](#create--select-a-firebase-project)
5. [Initialize Firebase in Your Project](#initialize-firebase-in--project)
6. [Build Your Project](#build--project)
7. [Deploy to Firebase Hosting](#deploy-to-firebase-hosting)
8. [Deploy Firestore Rules & Indexes](#deploy-firestore-rules--indexes)
9. [Deploy Cloud Functions](#deploy-cloud-functions)
10. [Deploy Storage Rules](#deploy-storage-rules)
11. [Deploy Everything at Once](#deploy-everything-at-once)
12. [Useful Firebase CLI Commands](#useful-firebase-cli-commands)
13. [Common Errors & Fixes](#common-errors--fixes)
14. [Quick Checklist](#quick-checklist)

---

## 1. Prerequisites

- A [Firebase account](https://console.firebase.google.com/) (use  Google account)
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

Login with  Google account from the terminal:

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
3. Once created, note  **Project ID**

To list projects from CLI:

```bash
firebase projects:list
```

---

## 5. Initialize Firebase in Your Project

Navigate to  project folder:

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

Deploy  site:

```bash
firebase deploy --only hosting
```

After deployment, you'll get a live URL like:

```
https://-project-id.web.app
```

---

## 8. Deploy Firestore Rules & Indexes

If  project uses Firestore:

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

*Last updated: keep this file versioned in  repo and update commands as Firebase CLI evolves.*
