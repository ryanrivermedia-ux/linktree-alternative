# Wave 1 Prerequisites — Manual Setup Checklist

Complete all steps before running `/karimo:run --prd admin-dashboard`.

---

## Step 1: Create Turso Account + Database

1. Go to [turso.tech](https://turso.tech) and create an account
2. Create a new database:
   ```bash
   turso db create linktree-admin
   ```
3. Get the database URL and auth token:
   ```bash
   turso db show linktree-admin --url      # → TURSO_DATABASE_URL
   turso db tokens create linktree-admin   # → TURSO_AUTH_TOKEN
   ```

---

## Step 2: Generate Session Secret

```bash
openssl rand -base64 32
# Copy output → SESSION_SECRET
```

---

## Step 3: Generate Admin Password Hash

```bash
node -e "const b = require('bcryptjs'); b.hash('YOUR_PASSWORD', 12).then(h => console.log(h))"
# Copy output → ADMIN_PASSWORD_HASH
```

Or use an online bcrypt generator with cost 12.

---

## Step 4: Set Local Environment Variables

Create `.env.local` in project root (already in `.gitignore`):

```bash
SESSION_SECRET=<from step 2>
ADMIN_PASSWORD_HASH=<from step 3>
TURSO_DATABASE_URL=<from step 1>
TURSO_AUTH_TOKEN=<from step 1>
```

---

## Step 5: Create Vercel Project

1. Go to [vercel.com](https://vercel.com) and import the GitHub repo (`opensesh/linktree-alternative`)
2. Set framework preset: **Next.js**
3. Do NOT set output directory — let Vercel auto-detect

---

## Step 6: Add Env Vars to Vercel

In Vercel project → Settings → Environment Variables, add all 4 vars:

| Name | Environments |
|---|---|
| `SESSION_SECRET` | Production, Preview |
| `ADMIN_PASSWORD_HASH` | Production, Preview |
| `TURSO_DATABASE_URL` | Production, Preview |
| `TURSO_AUTH_TOKEN` | Production, Preview |

---

## Step 7: Verify

- [ ] `.env.local` exists with all 4 vars
- [ ] `turso db show linktree-admin` returns database info
- [ ] Vercel project created and linked to repo
- [ ] All 4 env vars set in Vercel dashboard

---

Once all boxes are checked, authorize Wave 1:

```
/karimo:run --prd admin-dashboard
```
