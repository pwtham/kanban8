---
description: Publish this project to GitHub (push, README, About, Pages, CI/CD) with a sensitive-data check
argument-hint: <owner/repo or repo URL> [branch]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

Publish this project to GitHub. The repo info comes from the user: `$ARGUMENTS`

If `$ARGUMENTS` is empty or ambiguous, ask the user for the repo (`owner/name` or URL) and the default branch (assume `main`) before doing anything else. Normalize it to `owner/name`; the site URL is `https://<owner>.github.io/<name>/`.

Work through the steps in order. Report what you did and what you could not do at the end.

## 1. Sensitive-data check (must pass before any push)

Run this first and again right before pushing. Never push if it finds something unresolved.

- List what would be uploaded: `git ls-files` plus untracked, non-ignored files.
- Search those files (Grep) for: API keys/tokens (`ghp_`, `github_pat_`, `sk-`, `AKIA`, `xox[bp]-`), `password`/`secret`/`token` assignments, private keys (`BEGIN .* PRIVATE KEY`), `.env` files, `*.pem`/`*.key`/`id_rsa*`, and personal email addresses or phone numbers.
- Also check git history of files already committed (`git log -p -S<pattern>` for anything found).
- Project rule (see CLAUDE.md): the FormSubmit email address may appear only in `FORMSUBMIT_ENDPOINT` in `index.html` and nowhere else (README, workflow, commit messages included). Decide whether that endpoint is acceptable to publish and flag it to the user.
- Make sure a `.gitignore` exists covering `.env*`, keys, `.claude/settings.local.json`, OS/editor junk. Add it if missing.
- If anything sensitive is found: stop, show the file and line (redact the value), and ask the user how to proceed. Do not rewrite history or force-push without explicit approval.
- Check git identity (`git config user.name/email`); warn if a private email would be baked into commits.

## 2. README

Create or update `README.md` from the actual project contents (read the code, do not invent features): title, one-line description, live demo link (the Pages URL), features, how to run locally, tech/constraints, project structure, CI/CD note. Keep any existing hand-written sections that are still accurate. No secrets, no private emails.

## 3. GitHub Pages

- Static site: publish only the files the site needs (for this project `index.html`), not repo docs or config.
- Pages source must be **GitHub Actions** (Settings → Pages → Build and deployment → Source).

## 4. CI/CD via GitHub Actions

Create or update `.github/workflows/pages.yml`: trigger on push to the default branch and `workflow_dispatch`; permissions `contents: read`, `pages: write`, `id-token: write`; steps checkout → stage site into `_site` → `configure-pages` → `upload-pages-artifact` → `deploy-pages` (use current major versions). If the project gains tests or a linter, add a job that runs before deploy. Keep third-party actions to the official `actions/*` set.

## 5. Push

1. Show the user a summary: repo, branch, files to be committed, and the result of the sensitive-data check. Ask for confirmation before pushing.
2. Set up the remote: if `origin` exists and differs, ask before changing it. Prefer SSH (`git@github.com:owner/name.git`) if a key is configured, otherwise HTTPS.
3. Commit with a clear message, then `git push -u origin <branch>`. Never use `--force`.
4. If authentication fails, do not ask for or type passwords or tokens. Tell the user how to authenticate (SSH key, `gh auth login`, or a credential manager) and stop.

## 6. GitHub About section (description, topics, website)

Set the repo description (one sentence), topics (3–6, lowercase), and **website = the Pages URL** from step 3.

- If `gh` is installed and authenticated: `gh repo edit owner/name --description "..." --homepage "<pages-url>" --add-topic a,b,c`.
- Otherwise use the browser pane on `https://github.com/<owner>/<name>` (the gear icon next to "About"). The user must sign in themselves; never enter their credentials. If a browser isn't available, print the exact values for the user to paste.
- Enabling Pages (step 3) is a repo setting: if it cannot be done with `gh api` or the browser, give the user the exact click path.

## 7. Verify and report

- Confirm the workflow run succeeded (Actions tab or `gh run list`), and load the Pages URL to check it returns the site and not a 404. Allow about a minute after the run turns green.
- Confirm the About section shows the description, topics and the Pages link.
- Final report: what was done, what was verified, what the user still has to do by hand (for example signing in, enabling Pages), and anything the sensitive-data check flagged.
