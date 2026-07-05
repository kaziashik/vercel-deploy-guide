# Full Deployment Guide (Frontend: Firebase Hosting, Backend: Vercel)

``` text
                Development
+----------------+      HTTP      +------------------+
| React (Vite)   | <------------> | Express Backend  |
| localhost:5173 |                | localhost:5000   |
+----------------+                +------------------+

                    ↓ Deploy

+----------------------+   HTTPS   +----------------------+
| Firebase Hosting     | <-------> | Vercel Backend       |
| https://your.web.app |           | https://api.vercel.. |
+----------------------+           +----------------------+
```

## 1. Backend (Vercel)

### Environment variables

Set in Vercel: - MONGODB_URI - JWT_SECRET - FIREBASE\_\* (if used) -
CLIENT_URLS=https://your-app.web.app,https://your-app.firebaseapp.com

Your config should parse:

``` js
clientUrls: process.env.CLIENT_URLS.split(",")
```

CORS:

``` js
app.use(cors({
  origin: (origin, callback) => {
    if (!origin || config.clientUrls.includes(origin)) {
      return callback(null, true);
    }
    return callback(new Error("Not allowed by CORS"));
  },
  credentials: true,
}));
```

Deploy:

``` bash
npm install -g vercel
vercel
vercel --prod
```

Copy the backend URL, e.g.

`https://portfolio-backend.vercel.app`

------------------------------------------------------------------------

## 2. Frontend (Firebase Hosting)

Create `.env.production`

``` env
VITE_API_BASE_URL=https://portfolio-backend.vercel.app/api
VITE_CLOUDINARY_CLOUD_NAME=rnonsqcn
VITE_CLOUDINARY_UPLOAD_PRESET=portfolio_profile
```

Use it everywhere:

``` js
const API = import.meta.env.VITE_API_BASE_URL;
fetch(`${API}/profile`);
```

Never hardcode `http://localhost:5000`.

Deploy:

``` bash
npm install -g firebase-tools
firebase login
firebase init hosting
npm run build
firebase deploy
```

------------------------------------------------------------------------

## 3. Request Flow

``` text
Browser
   │
   ▼
Firebase Hosting
   │
fetch(VITE_API_BASE_URL)
   │
   ▼
Vercel Express API
   │
MongoDB Atlas
```

------------------------------------------------------------------------

## 4. Local vs Production

Local `.env`

``` env
VITE_API_BASE_URL=http://localhost:5000/api
```

Production `.env.production`

``` env
VITE_API_BASE_URL=https://portfolio-backend.vercel.app/api
```

Vite automatically uses the correct file.

------------------------------------------------------------------------

## 5. Checklist

-   MongoDB Atlas accessible
-   Backend deployed to Vercel
-   CLIENT_URLS updated
-   Frontend uses VITE_API_BASE_URL
-   Firebase Hosting deployed
-   Test login, profile, contact form and uploads

That's the complete workflow for future deployments.
