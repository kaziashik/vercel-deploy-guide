# Deployment Guide — Portfolio Backend (Vercel) + Frontend (Firebase Hosting)

> Read this once, deploy confidently every time after. Keep it in your repo.

---

## 0. The Big Picture First

Right now, everything talks to everything over `localhost`. Once deployed, you have **two separately-hosted things** that must be told about each other's real internet address:

```
┌─────────────────────────┐         ┌──────────────────────────┐
│   Firebase Hosting        │         │   Vercel                  │
│   (your React frontend,   │  HTTPS  │   (your Express backend,  │
│   static files only)      │ ──────> │   /api/... routes)        │
│   kazi-portfolio.web.app  │ <────── │   portfolio-backend       │
└─────────────────────────┘  JSON    │   -xyz.vercel.app         │
                                       └──────────────────────────┘
                                                  │
                                                  │ MongoDB
                                                  │ connection
                                                  ▼
                                       ┌──────────────────────────┐
                                       │   MongoDB Atlas            │
                                       │   (your database, already │
                                       │   set up)                  │
                                       └──────────────────────────┘
```

**Two things must match up, or nothing works:**
1. Your **frontend** must know the backend's real URL (not `localhost:5000`) → set via `VITE_API_BASE_URL`
2. Your **backend** must allow requests *from* the frontend's real URL → set via `CLIENT_URLS` (this is exactly the `cors()` code you pasted)

Everything below is really just: deploy backend → get its URL → tell frontend → deploy frontend → get its URL → tell backend → redeploy backend once. After that, it's just "push code, redeploy" forever.

---

## 1. Deploy the Backend to Vercel

### 1.1 Push your backend to GitHub
If it isn't already:
```bash
cd portfolio-backend
git init
git add .
git commit -m "Initial commit"
```
Create a new repo on GitHub, then:
```bash
git remote add origin https://github.com/<your-username>/portfolio-backend.git
git push -u origin main
```

### 1.2 Import it into Vercel
1. Go to [vercel.com](https://vercel.com) → **Add New** → **Project**
2. Import your `portfolio-backend` GitHub repo
3. Vercel auto-detects `vercel.json` (already in your project) — you don't need to change build settings

### 1.3 Add every environment variable
This is the step people forget — **Vercel does not read your `.env` file** (it's gitignored, so it never even reaches Vercel). You must paste each value manually.

Go to your Vercel project → **Settings** → **Environment Variables**, and add every single one of these:

| Key | Value |
|---|---|
| `DATABASE_URL` | your real MongoDB Atlas connection string |
| `JWT_ACCESS_SECRET` | your real secret |
| `JWT_REFRESH_SECRET` | your real secret |
| `JWT_ACCESS_EXPIRES_IN` | `1d` |
| `JWT_REFRESH_EXPIRES_IN` | `7d` |
| `BCRYPT_SALT_ROUNDS` | `12` |
| `ADMIN_EMAIL` | your admin email |
| `ADMIN_PASSWORD` | (only needed if you re-run the seed script against production — see 1.6) |
| `NODE_ENV` | `production` |
| `CLIENT_URLS` | *leave a placeholder for now* — `http://localhost:5173` — you'll update this in Step 3 |

Apply each to **Production**, **Preview**, and **Development** environments (Vercel asks you to pick which).

### 1.4 Deploy
Click **Deploy**. Vercel builds and gives you a URL like:
```
https://portfolio-backend-yourname.vercel.app
```
**Copy this URL — you'll need it in Step 2.**

### 1.5 Confirm MongoDB Atlas allows Vercel in
Vercel's serverless functions don't have one fixed IP address. In Atlas → **Network Access**, confirm `0.0.0.0/0` is listed and **Active** (you already did this earlier — just double-check it's still there).

### 1.6 Seed your admin account on production
Your local `npm run seed:admin` only touched your **local** database connection. If your production `DATABASE_URL` points to the same Atlas cluster/database, you're already done. If it's a different database, run the seed script once with production values:
```bash
DATABASE_URL="<production-url>" ADMIN_EMAIL="..." ADMIN_PASSWORD="..." node scripts/seedAdmin.js
```

### 1.7 Test it before moving on
```
GET https://portfolio-backend-yourname.vercel.app/health
```
Should return `{ "success": true, "message": "OK", ... }`. Also try `/api/profile` — if it 500s, check the Vercel deployment's **Logs** tab (usually a missing/misspelled env var).

---

## 2. Deploy the Frontend to Firebase Hosting

### 2.1 The one concept that trips people up

```
┌──────────────────────────────────────────────────────────────┐
│  BUILD TIME (on your machine, when you run `npm run build`)   │
│                                                                  │
│   .env file  ──read once──>  Vite  ──bakes values into──>      │
│   VITE_API_BASE_URL=...              the JS files in dist/     │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│  RUNTIME (in a visitor's browser, after Firebase serves it)     │
│                                                                  │
│   The JS already has the URL hardcoded in it — Firebase          │
│   Hosting is 100% static files, there is no "runtime env"        │
│   to read from at all.                                           │
└──────────────────────────────────────────────────────────────┘
```

**This means:** if you `firebase deploy` without first putting the *real* Vercel URL in `.env` and rebuilding, your live site will silently keep calling `localhost:5000` — which doesn't exist on the internet, and every API call will just fail. **Always update `.env` and re-run `npm run build` before every `firebase deploy`.**

### 2.2 Update your frontend `.env` with the real backend URL
```dotenv
VITE_API_BASE_URL=https://portfolio-backend-yourname.vercel.app
```
(Use the exact URL you got in Step 1.4.)

### 2.3 Install the Firebase CLI (one-time, on your machine)
```bash
npm install -g firebase-tools
firebase login
```

### 2.4 Initialize Firebase Hosting in your frontend project (one-time)
```bash
cd portfolio-frontend
firebase init hosting
```
Answer the prompts:
- **"Use an existing project"** → pick the Firebase project you already created for Auth
- **"What do you want to use as your public directory?"** → type `dist`
- **"Configure as a single-page app?"** → **Yes** (critical — without this, refreshing any route other than `/` will 404)
- **"Set up automatic builds with GitHub?"** → No, for now (keep it manual until you're comfortable)
- **"File dist/index.html already exists. Overwrite?"** → No (if asked — you haven't built yet, this may not appear)

This creates `firebase.json` and `.firebaserc` in your project — commit both to git.

### 2.5 Build with the real backend URL baked in
```bash
npm run build
```

### 2.6 Deploy
```bash
firebase deploy --only hosting
```
You'll get a URL like:
```
https://kazi-portfolio.web.app
```
**Copy this — you need it for Step 3.**

---

## 3. Close the Loop — Tell the Backend About the Frontend

This is the step that makes the CORS code you pasted actually let requests through.

### 3.1 Update `CLIENT_URLS` in Vercel
Go back to Vercel → your backend project → **Settings** → **Environment Variables** → edit `CLIENT_URLS`:
```
https://kazi-portfolio.web.app,https://kazi-portfolio.firebaseapp.com,http://localhost:5173
```
(Firebase Hosting gives you *two* working domains for the same project — `.web.app` and `.firebaseapp.com` — include both. Keep `localhost:5173` too, so local development still works.)

### 3.2 Redeploy the backend
Environment variable changes in Vercel **don't apply retroactively** — you must trigger a new deployment:
- Easiest: Vercel dashboard → **Deployments** tab → click the **⋯** on the latest deployment → **Redeploy**
- Or: push any small commit to your GitHub repo (Vercel auto-deploys on every push)

### 3.3 Add the Firebase Hosting domain to Firebase Auth's allowed list
Firebase Authentication separately checks *which domains are allowed to use it* (unrelated to your backend's CORS list).

Firebase Console → **Authentication** → **Settings** tab → **Authorized domains**. Your `.web.app` domain is usually added automatically — double check it's there. If you later add a custom domain (e.g. `kazi.com`), add it here too, or Google/email login will fail with an `auth/unauthorized-domain` error.

---

## 4. End-to-End Test Checklist

Run through this on the **real deployed frontend URL**, not localhost:

- [ ] Homepage loads, profile/projects/experience sections show real data (not stuck on loading spinners)
- [ ] Open browser DevTools → Network tab → confirm requests go to your Vercel URL, not `localhost`
- [ ] No red CORS errors in the console
- [ ] Log in as admin (email/password and Google both)
- [ ] Add/update/delete a project — confirm it reflects immediately
- [ ] Contact form sends successfully
- [ ] View CV / Update CV both work

---

## 5. Your Ongoing Workflow (Once Set Up)

```
Backend change:
  edit code → git push → Vercel auto-deploys → done

Frontend change:
  edit code → update .env if API URL ever changes → npm run build → firebase deploy → done
```

```
┌─────────────┐     git push      ┌──────────────┐
│  Edit backend│ ────────────────> │ Vercel auto-  │
│  code        │                    │ redeploys     │
└─────────────┘                    └──────────────┘

┌─────────────┐   npm run build   ┌──────────────┐  firebase deploy  ┌──────────────┐
│  Edit frontend│ ───────────────> │ dist/ folder  │ ─────────────────> │ Firebase       │
│  code         │                   │ regenerated   │                    │ Hosting live   │
└─────────────┘                   └──────────────┘                    └──────────────┘
```

---

## 6. Troubleshooting — Most Common First-Deploy Issues

| Symptom | Cause | Fix |
|---|---|---|
| CORS error in browser console on the live site | `CLIENT_URLS` on Vercel doesn't include your `.web.app` URL, or you forgot to redeploy after changing it | Re-check Step 3.1 and 3.2 |
| Live site still calls `localhost:5000` | You ran `firebase deploy` without rebuilding after updating `.env` | Redo Step 2.5 then 2.6 |
| Google/email login fails with `auth/unauthorized-domain` | Firebase Hosting domain missing from Authorized Domains | Step 3.3 |
| Backend works via Postman but not from the deployed frontend | Almost always CORS — check the exact origin Vercel is rejecting in the browser's error message and compare it character-for-character to `CLIENT_URLS` | Fix the mismatch, redeploy backend |
| Admin login works locally but refresh token doesn't persist on deployed site | Cross-domain cookie needs `SameSite=None; Secure=true` — this is already handled in `auth.controller.js` based on `NODE_ENV`, so confirm `NODE_ENV=production` is actually set in Vercel | Add/fix the env var, redeploy |
| `/api/profile` returns 500 on Vercel but works locally | Usually a missing environment variable in Vercel (typo'd key name, or forgot to add it to "Production" scope) | Vercel → Deployments → click the failing one → **Logs** |
| MongoDB connection times out only on Vercel | Atlas Network Access doesn't include `0.0.0.0/0` | Step 1.5 |

---

## 7. Reference — Where Each Variable Lives

```
┌───────────────────────────┐        ┌───────────────────────────┐
│ Vercel (backend)            │        │ Your local .env (frontend)  │
│ Settings → Env Variables    │        │ read once at build time      │
│                              │        │                               │
│ DATABASE_URL                │        │ VITE_API_BASE_URL            │
│ JWT_ACCESS_SECRET            │        │ VITE_FIREBASE_*               │
│ JWT_REFRESH_SECRET           │        │ VITE_CLOUDINARY_*             │
│ BCRYPT_SALT_ROUNDS           │        │                               │
│ ADMIN_EMAIL / ADMIN_PASSWORD │        │ (baked into dist/ JS files,   │
│ CLIENT_URLS  ← the CORS list │        │  Firebase Hosting never sees  │
│ NODE_ENV=production          │        │  or stores these directly)    │
└───────────────────────────┘        └───────────────────────────┘
```

Firebase Hosting itself has **no environment variables at all** — it only ever serves the static files inside `dist/`, exactly as they were built. Any secret that needs to stay secret (JWT secrets, database credentials) belongs on Vercel, never in the frontend's `.env` — anything prefixed `VITE_` ends up readable in your published JS bundle, which is fine for things like public API URLs and Firebase's public config, but never acceptable for real secrets.

---

## 8. Cross-Checking a Generic Vercel Deployment Checklist Against This Project

You may run into deployment checklists/tutorials online (common in bootcamp courses) that list steps like "comment out `client.close()`", "ping MongoDB on startup", etc. Here's what applies to **this specific project** and what doesn't, so you don't waste time chasing a fix for a problem this project doesn't have.

### 8.1 The build-time vs. runtime concept (the one that trips everyone up)

```
┌──────────────────────────────────────────────────────────────┐
│  BUILD TIME (on your machine, when you run `npm run build`)   │
│                                                                  │
│   .env file  ──read once──>  Vite  ──bakes values into──>      │
│   VITE_API_BASE_URL=...              the JS files in dist/     │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│  RUNTIME (in a visitor's browser, after Firebase serves it)     │
│                                                                  │
│   The JS already has the URL hardcoded in it — Firebase          │
│   Hosting is 100% static files, there is no "runtime env"        │
│   to read from at all.                                           │
└──────────────────────────────────────────────────────────────┘
```
Practical consequence: if you change `.env` but forget to re-run `npm run build` before `firebase deploy`, the live site keeps using the **old** baked-in value — updating `.env` alone does nothing until you rebuild.

### 8.2 "Comment out `client.close()` / startup ping" — does NOT apply here

Some course templates connect to MongoDB using the **raw native driver**, ping it once at startup, and close the connection after each request. That pattern causes gateway timeouts on Vercel because every cold start has to reconnect from scratch, and a closed connection mid-request can hang the function.

**This project never does that.** `src/config/db.js` uses Mongoose with a cached connection stored on `global`, specifically to avoid this class of bug:

```
❌ ANTI-PATTERN (raw driver, closes the connection every time)
┌───────────────────────────────┐
│ Cold start → connect + ping     │
│ Handle request                   │
│ client.close()  ← closes it!     │
└───────────────────────────────┘
              │
              ▼ next request arrives
┌───────────────────────────────┐
│ No connection left open          │
│ Must reconnect from scratch       │
│ Slow — sometimes exceeds the       │
│ function's timeout entirely        │
└───────────────────────────────┘

✅ THIS PROJECT'S PATTERN (cached Mongoose connection)
┌───────────────────────────────┐
│ Cold start → connect once         │
│ Cache it on global._mongoose...    │
│ Handle request                     │
│ (connection is never closed)        │
└───────────────────────────────┘
              │
              ▼ next request (same warm function instance)
┌───────────────────────────────┐
│ Reuse the cached connection         │
│ Instant — no reconnect delay         │
└───────────────────────────────┘
```
There is nothing to comment out in this project for this issue — it was designed around it from the start.

### 8.3 Firebase service-account base64 encoding — genuinely useful, check if you need it

If your own `/api/auth/firebase-login` route initializes the Firebase Admin SDK using a raw multi-line service-account JSON as an environment variable, Vercel can mangle the newlines inside it and break `JSON.parse`. The fix is to base64-encode it once, locally:

```
┌─────────────────────────┐   base64 encode    ┌──────────────────────────┐
│ firebase-admin-key.json    │ ──────────────────> │ one long base64 string     │
│ (multi-line JSON file,      │  (run locally once)  │ (safe to paste anywhere)   │
│  never committed to git)    │                       │                            │
└─────────────────────────┘                        └──────────────────────────┘
                                                                │
                                                paste into a Vercel env var
                                                                ▼
                                                      ┌──────────────────────────┐
                                                      │ FIREBASE_SERVICE_          │
                                                      │ ACCOUNT_BASE64               │
                                                      │ (single line, newlines        │
                                                      │  can no longer break it)       │
                                                      └──────────────────────────┘
                                                                │
                                                    decoded back to JSON at runtime
                                                                ▼
                                                      ┌──────────────────────────┐
                                                      │ Firebase Admin SDK            │
                                                      │ initializes correctly          │
                                                      └──────────────────────────┘
```

Locally, generate the string once:
```bash
node -e "console.log(Buffer.from(require('fs').readFileSync('./firebase-admin-key.json','utf8')).toString('base64'))"
```
Paste the output into Vercel as `FIREBASE_SERVICE_ACCOUNT_BASE64`. In your route, decode it back:
```js
const decoded = Buffer.from(process.env.FIREBASE_SERVICE_ACCOUNT_BASE64, "base64").toString("utf8");
const serviceAccount = JSON.parse(decoded);
```
Only relevant if your `firebase-login` implementation currently passes the raw JSON directly — worth checking if you ever see Firebase Admin initialization errors specifically on Vercel but not locally.

### 8.4 Quick cross-check table

| Checklist item | Status in this project |
|---|---|
| `vercel.json` present | ✅ Already there (`builds` + `rewrites` format) |
| `"start": "node ..."` script | ✅ `"start": "node src/server.js"` |
| Use `process.env.PORT` | ✅ Already in `config/index.js` |
| Whitelist `0.0.0.0/0` in Atlas | ✅ Confirmed active |
| Don't close the DB connection after each request | ✅ Never closes it — cached on `global` instead |
| Comment out `client.close()` / startup ping | ⛔ N/A — this project doesn't use that raw-driver pattern at all |
| Base64-encode Firebase service account key | ⚠️ Only needed if your `firebase-login` route reads a raw JSON env var — check your own implementation |
| No trailing slash in `CLIENT_URLS` entries | ⚠️ Double-check when you fill this in — `https://site.web.app` not `https://site.web.app/` |
| `sameSite` cookie value in dev | Either `"lax"` (what we built) or `"strict"` both work here — not a bug either way |

