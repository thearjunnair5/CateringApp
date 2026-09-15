# Deploying CaterFlow to Netlify

## What you need

| Requirement | Notes |
|---|---|
| **Netlify account** | Free tier is fine — https://app.netlify.com |
| **PostgreSQL database** | Not included. Use [Neon](https://neon.tech) or [Supabase](https://supabase.com) (both have free tiers) |
| **GitHub / GitLab account** | For the recommended Git-based deploy |

---

## Step 1 — Get a PostgreSQL database

1. Sign up at [Neon](https://neon.tech) (easiest free option).
2. Create a project and copy the **Connection string** — it looks like:
   ```
   postgresql://user:password@ep-xxx.us-east-1.aws.neon.tech/neondb?sslmode=require
   ```
3. Keep this string handy — you'll paste it into Netlify in Step 3.

---

## Step 2 — Push the code to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
gh repo create caterflow --public --push
# or push manually to your GitHub account
```

---

## Step 3 — Connect to Netlify

1. Go to **https://app.netlify.com** → **Add new site → Import an existing project**.
2. Choose **GitHub** and select your repository.
3. Netlify auto-detects `netlify.toml`. Leave all build settings as-is.
4. Before deploying, go to **Site configuration → Environment variables** and add:

   | Key | Value |
   |---|---|
   | `DATABASE_URL` | Your PostgreSQL connection string from Step 1 |

5. Click **Deploy site**.

---

## Step 4 — Run the database migration

After the first deploy, your database schema needs to be pushed once:

```bash
# From your local machine (with DATABASE_URL set in your terminal):
export DATABASE_URL="postgresql://..."
pnpm --filter @workspace/db run push
```

This creates all the tables (categories, ingredients, dishes, orders, etc.) and seeds the default categories.

---

## How it works on Netlify

```
Browser → GET /          → Netlify CDN → netlify-dist/public/index.html
Browser → GET /dishes    → Netlify CDN → netlify-dist/public/index.html  (SPA)
Browser → POST /api/... → Netlify Function → netlify-dist/functions/api.js → PostgreSQL
```

The `netlify.toml` at the root configures:
- **Build command**: `node scripts/netlify-build.mjs`
- **Publish directory**: `netlify-dist/public` (the React app)
- **Functions directory**: `netlify-dist/functions` (the Express API)
- **Redirects**: `/api/*` → function, `/*` → `index.html` (SPA fallback)
