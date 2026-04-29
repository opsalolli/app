# PowerWash CRM — Vercel Edition

Drop this folder into Vercel → click Deploy → done.

## What gets deployed

- **Frontend** (Expo Router → web) compiled to a static site
- **Backend** (FastAPI) running as Vercel Python serverless functions under `/api/*`
- A single `vercel.json` glues them together so `/api/*` hits the Python function and everything else serves the static web app

## Required environment variables (Vercel dashboard → Project → Settings → Environment Variables)

| Name | Value | Required |
|------|-------|----------|
| `MONGO_URL` | Your MongoDB Atlas connection string e.g. `mongodb+srv://user:pass@cluster.mongodb.net` | ✅ Yes |
| `DB_NAME` | `powerwash_crm` (or whatever you like) | ✅ Yes |
| `JWT_SECRET` | Any long random string | ✅ Yes |
| `STRIPE_API_KEY` | `sk_test_...` or `sk_live_...` for invoice payments | ⛔️ Optional |

> 💡 **MongoDB**: Get a **free** cluster at [mongodb.com/atlas](https://www.mongodb.com/atlas) (M0 = forever free). Whitelist `0.0.0.0/0` so Vercel can reach it.

## Deploy in 60 seconds

### Option A — drag and drop (no Git)
1. Unzip this archive
2. `npm i -g vercel` then run `vercel` from inside the folder, or
3. Open [vercel.com/new](https://vercel.com/new) → click **"Upload"** → drop the unzipped folder
4. Add the env vars above when prompted, click Deploy

### Option B — push to GitHub then import
```bash
unzip powerwash-crm-vercel.zip
cd powerwash-crm-vercel
git init && git add . && git commit -m "init"
git remote add origin https://github.com/YOU/powerwash-crm.git
git push -u origin main
```
Then go to [vercel.com/new](https://vercel.com/new) → import the repo → add env vars → Deploy.

That's it. Vercel will:
1. Run `yarn install && expo export --platform web` to build the static site into `/public`
2. Detect `api/index.py`, install `requirements.txt`, deploy as a Python serverless function
3. Serve everything at one domain (`https://your-app.vercel.app`)

## Project structure

```
.
├── api/
│   ├── index.py            # Vercel entrypoint — re-exports FastAPI app
│   ├── server.py           # Full backend (1000+ lines)
│   └── requirements.txt
├── frontend/
│   ├── app/                # Expo Router screens (file-based routing)
│   ├── src/                # auth.tsx, theme.ts
│   ├── package.json
│   └── ...
├── requirements.txt        # duplicated at root for Vercel auto-detection
├── vercel.json             # routing + build config
├── .gitignore
└── README.md
```

## Local development

Backend:
```bash
cd api
pip install -r requirements.txt
# point MONGO_URL etc. at your local mongo
uvicorn server:app --reload --port 8001
```

Frontend:
```bash
cd frontend
yarn
EXPO_PUBLIC_BACKEND_URL=http://localhost:8001 npx expo start
```

## Native mobile build

The `frontend/` folder is a regular Expo project. To build the iOS/Android app:
```bash
cd frontend
yarn
npx expo run:ios       # or run:android
# or, with EAS:
npx eas build --platform all
```

Set the production backend URL via `EXPO_PUBLIC_BACKEND_URL=https://your-app.vercel.app` before building.

## Caveats with Vercel serverless

- **Cold starts** — first request after idle takes ~1-2s while the Python function spins up.
- **30s execution limit** (set in `vercel.json`). All current endpoints finish in <500 ms.
- **MongoDB connection pooling** — Motor reconnects per cold start. For higher traffic, switch to MongoDB Atlas Data API or upgrade to Vercel's Edge Runtime.
- **Stripe webhooks** — point them at `https://your-app.vercel.app/api/webhook/stripe`.

## License

MIT
