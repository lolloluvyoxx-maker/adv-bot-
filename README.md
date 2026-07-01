# ZyroX Discord Bot — Railway Deployment

This repo is set up to deploy straight from GitHub to [Railway](https://railway.app).

## What was changed to make this deployable

- **`requirements.txt`** cleaned up — removed stdlib fakes (`asyncio`, `typing`, `pathlib`, `datetime`,
  `collection`) and the conflicting `discord` package that clashes with `discord.py`, plus unused
  heavy/broken deps (`Quart`, `pymongo`, `motor`, `tasksio`, `Augmentor`, `pyttsx3`).
- **Secrets moved to environment variables**: the Pexels and Giphy API keys were hardcoded in
  `cogs/commands/image.py` and `cogs/commands/fun.py` — they now read from `PEXELS_API_KEY` /
  `GIPHY_API_KEY`. The Discord bot token and Groq AI key already used env vars (`TOKEN`,
  `GROQ_API_KEY`); see `.env.example`.
- **Keep-alive server** now binds to Railway's `PORT` env var instead of a hardcoded port, so
  Railway's health checks work.
- **SQLite files consolidated** under `db/` (two stray files, `rr.db` and `j2c_data.db`, were at the
  project root — moved so a single Volume mount covers everything). The `db/` folder is created
  automatically on startup.
- Added `Procfile`, `railway.json`, `.python-version`, and `.gitignore`.

## 1. Push this to GitHub

```bash
cd ZyroX
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

`.env` and all `*.db` files are gitignored on purpose — never commit real secrets or local bot data.

## 2. Create the Railway project

1. Go to [railway.app](https://railway.app) → **New Project** → **Deploy from GitHub repo**.
2. Select this repository. Railway will detect Python via Nixpacks automatically and pick up
   `railway.json` / `Procfile` for the start command (`python CodeX.py`).

## 3. Set environment variables

In the Railway service → **Variables**, add:

| Variable | Required | Notes |
|---|---|---|
| `TOKEN` | ✅ | Discord bot token — Developer Portal → Bot → Reset Token |
| `GROQ_API_KEY` | optional | Needed for the AI chat commands |
| `PEXELS_API_KEY` | optional | Needed for image search commands |
| `GIPHY_API_KEY` | optional | Needed for GIF commands |

> The bot token that was in the uploaded `.env` was already a placeholder (masked with `x`s), but
> as a general rule: if a real token is ever exposed anywhere (git history, screenshots, chat), treat
> it as compromised and regenerate it in the Discord Developer Portal immediately.

## 4. Add a persistent Volume (important)

This bot stores all its data in local SQLite files under `db/`. **Railway's filesystem is ephemeral** —
without a Volume, every redeploy wipes levels, warns, tickets, giveaways, etc.

1. In the Railway service → **Settings → Volumes** → **New Volume**.
2. Mount path: `/app/db`
3. Redeploy. From then on, everything in `db/` persists across deploys.

## 5. Deploy

Railway deploys automatically on push to `main`. Watch the **Deployments** tab for build/runtime logs.
Once running, you should see the ZyroX ASCII banner in the logs and the bot come online in Discord.

## Notes

- The bot also starts a tiny Flask server (used for Railway's health check / uptime ping) on the
  `PORT` Railway provides — no action needed, it's automatic.
- `config.yml` holds non-secret bot behavior settings (AI model, presence messages, word filters,
  etc.) — edit and commit it directly rather than using env vars.
