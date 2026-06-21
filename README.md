# Bundler vs No-Bundler Vercel Deploy — Quick Comparison

| | `With_Bundler_vercel-deploy-guide.md` | `With_No_Bundler_vercel_deploy_guide.md` |
|---|---|---|
| **Language** | TypeScript | Plain JavaScript |
| **Build step** | Required (`tsup`) | None |
| **Entry point** | `dist/server.js` (built) | `index.js` (raw source) |
| **Covers** | Compiling/bundling correctly | Keeping a live app stable on Vercel |
| **Database** | Not covered | MongoDB Atlas (connection handling, IP whitelist) |
| **Auth** | Not covered | Firebase Admin key + JWT in httpOnly cookies |
| **CORS / cookies** | Not covered | Production CORS + `sameSite`/`secure` config |

## Which one to follow

- **Writing TypeScript?** → Follow `With_Bundler` for the build/tsup/vercel.json setup.
- **Plain JavaScript, no build step?** → Follow `With_No_Bundler`.
- **Using MongoDB and/or Firebase auth (either language)?** → Also apply the MongoDB connection-handling and auth/cookie steps from `With_No_Bundler` — those issues aren't about TS vs JS, they're about serverless + DB/auth, so they apply either way.

**Rule of thumb:** `With_Bundler` = how to compile correctly. `With_No_Bundler` = how to not break once it's running. A TS + MongoDB + Firebase project needs both.
