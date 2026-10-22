# Setting up the status monitor (occult-lab/occult-status)

This repository is ready: Upptime's workflows (template v1.46.3), our
`.upptimerc.yml` (13 checks across the three products) and a README. It only
needs a token and a push. Steps are from the Upptime docs
(https://upptime.js.org/docs/get-started, /configuration, /notifications) and
the template's own workflow files.

Do steps 1–2 **before** pushing: the push of `.upptimerc.yml` starts the
"Setup CI" workflow straight away, and without the token it runs with GitHub's
default token, which cannot update workflow files. Push only after the backend
with `/api/health/` is deployed, or the API checks start as "down".

## 1. Create the token

GitHub → Settings → Developer settings → Personal access tokens →
**Fine-grained tokens** → Generate new token.

- **Token name:** `occult-status upptime`
- **Expiration:** 90 days or longer (Upptime's docs suggest 90 days or more;
  when it expires the checks stop, so set a reminder to renew it).
- **Resource owner:** `occult-lab` (the organisation must allow fine-grained
  tokens; an org owner may need to approve the request).
- **Repository access:** Only select repositories → `occult-lab/occult-status`.
- **Repository permissions — Read and write for exactly these four:**
  - Actions
  - Contents
  - Issues
  - Workflows

  (Metadata: read-only is added automatically.) No organisation permissions.

The docs only document a fine-grained token. A GitHub App works too
(`GH_APP_ID` variable + `GH_APP_PRIVATE_KEY` secret, same four permissions),
but a token is simpler.

## 2. Add the secrets

Repository → Settings → Secrets and variables → **Actions** → New repository
secret. Paste values with no spaces or line breaks around them.

| Secret | Value | Needed |
|---|---|---|
| `GH_PAT` | the token from step 1 | yes |

No other secret is needed: every check is a public URL. (The API health check
also runs a real calculation through the engine, without an API key, so nothing
is charged.)

**Notifications (optional).** Upptime alerts through the providers below when
their secrets are set (all are already in the config's `secrets:` allowlist; a
provider must be listed there to work):

- Email (SMTP): `NOTIFICATION_EMAIL=true`, `NOTIFICATION_EMAIL_SMTP=true`,
  `NOTIFICATION_EMAIL_FROM`, `NOTIFICATION_EMAIL_TO`,
  `NOTIFICATION_EMAIL_SMTP_HOST`, `NOTIFICATION_EMAIL_SMTP_PORT`,
  `NOTIFICATION_EMAIL_SMTP_USERNAME`, `NOTIFICATION_EMAIL_SMTP_PASSWORD`
- Slack: `NOTIFICATION_SLACK=true`, `NOTIFICATION_SLACK_WEBHOOK_URL`
- Discord: `NOTIFICATION_DISCORD_WEBHOOK_URL`
- Telegram: `NOTIFICATION_TELEGRAM=true`, `NOTIFICATION_TELEGRAM_BOT_KEY`,
  `NOTIFICATION_TELEGRAM_CHAT_ID`

To use another provider from https://upptime.js.org/docs/notifications, add its
secret names to `secrets:` in `.upptimerc.yml` as well.

## 3. Push, then check Actions

1. Push this repository (`git push -u origin main`).
2. Repository → **Actions**: if GitHub shows "Workflows aren't being run on this
   repository", click **I understand my workflows, go ahead and enable them**.
3. The push of `.upptimerc.yml` runs **Setup CI** by itself. If it did not run
   (for example Actions were still disabled), open Actions → Setup CI →
   **Run workflow** once. It regenerates the workflows from our config, writes
   the first response times and README table, and builds the graphs.
4. After that, **Uptime CI** runs every 5 minutes on its own. Within ~10 minutes
   `history/summary.json` exists and the three status pages show live data
   once their sites have `STATUS_REPO=occult-lab/occult-status` and
   `STATUS_BRANCH=main` (Vercel env, all three projects).

Scheduled runs can start a few minutes late when GitHub is busy; that is
normal for GitHub Actions and is why each page shows "last checked".

## GitHub Pages (optional)

Not needed: our three sites read this repository's files directly.

- Without Pages the workflows do **not** fail. "Setup CI" and the daily "Static
  Site CI" still build Upptime's own status site and push it to a `gh-pages`
  branch; with Pages off that branch is simply never served.
- To skip that daily build entirely: Actions → **Static Site CI** → ⋯ →
  **Disable workflow**. (This is a repository setting, so Upptime's weekly
  template update does not undo it. Setup CI still builds the site when the
  config changes; that is harmless.)
- To publish Upptime's page as well: Settings → Pages → Deploy from a branch →
  `gh-pages` / root. It appears at https://occult-lab.github.io/occult-status/.

## What is monitored

13 checks, every 5 minutes. Slugs are fixed in `.upptimerc.yml` and the sites
group them with `status-groups.json`:

- Occult API: `occult-api` (health, including a real calculation through the
  engine), `occult-api-website`, `occult-api-docs`, `occult-api-mcp` (the
  catalogue the MCP server searches; the server itself runs on users' machines)
- Yogatara: `yogatara-website`, `yogatara-api` (health), `yogatara-login`
  (expects 401 when signed out), `yogatara-ai` (AI service reachability, no
  model call), `yogatara-courses`, `yogatara-community`
- Yoga Panchang: `yogapanchang-website`, `yogapanchang-daily-panchang`,
  `yogapanchang-calculators`

Admin and payment endpoints are deliberately not monitored publicly.

Outages become issues labelled `status` and the site's slug; they close when
the site recovers (Upptime deletes outages shorter than 15 minutes instead of
closing them).
