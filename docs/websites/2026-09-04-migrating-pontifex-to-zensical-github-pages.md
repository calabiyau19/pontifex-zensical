## Migrating Pontifex from Jekyll to Zensical, Live on GitHub Pages

<!-- Hero image suggestion: side-by-side of the old Jekyll/Chirpy homepage and the new Zensical card-grid homepage, dark/teal color scheme -->

This documents the full path from a locally-running Zensical site to `pontifex.site` being served entirely by Zensical via GitHub Pages, replacing the original Jekyll/Chirpy setup.

---

## Starting Point

Zensical had been running locally on `zensical-vm` (192.168.1.144) via `zensical.service`, with content already migrated from Jekyll. The original Jekyll site remained live at `pontifex.site`, served from the `calabiyau19.github.io` GitHub repo. The goal was to retire Jekyll and have Zensical take over the domain.

---

## Step 1 — Deciding on a New Repo, Not Reusing the Old One

Two options existed: reuse `calabiyau19.github.io` directly, or create a new repo and move the custom domain over once proven. Reusing the existing repo would have meant force-pushing Zensical's unrelated git history over Jekyll's, and `pontifex.site` would be live and at risk the moment anything was pushed — no safety net if the build failed.

The safer path was chosen: a new repo, tested completely at its own throwaway URL first, with `pontifex.site` only touched after a clean build was confirmed.

**Key fact confirmed from GitHub's docs:** custom domain assignment is a *repository settings* operation, entirely separate from DNS and from the repo's name. This meant the new repo didn't need to be named anything special — `pontifex-zensical` was fine.

---

## Step 2 — Verifying DNS Before Touching Anything

`pontifex.site`'s DNS is hosted on Cloudflare (registrar: Namecheap). Checking the actual DNS records confirmed:

- Three `A` records on the apex domain pointing to GitHub Pages' standard IPs, set to **DNS only** (not proxied)
- A `CNAME` on `www` pointing to `calabiyau19.github.io`

These records point at GitHub's generic, shared Pages infrastructure — not at any specific repo — so **no DNS changes were needed at any point** in this migration.

---

## Step 3 — Checking for Sensitive Content Before Going Public

Before creating any git history, all posts were audited for content that should stay local-only, matching the same pattern used on the Jekyll site. Eight posts and one screenshot were identified as sensitive (home-network details) and excluded via `.gitignore` rather than deleted, so they'd remain fully functional on the local site but never reach the public repo.

A second pass caught leftover AI-generated "hero image suggestion" comment placeholders left in a few posts — cleaned up before anything was committed.

---

## Step 4 — Setting Up Git on `zensical-vm`

```sh
cd ~/websites/zensical && git init
```

```sh
git config user.name "calabiyau19"
```

```sh
git config user.email "calabiyau19@gmail.com"
```

A `.gitignore` was created covering the standard build artifacts (`.venv/`, `site/`, `.cache/`, `__pycache__/`) plus the eight sensitive posts and the one sensitive image.

```sh
git add . && git status
```

Status was checked carefully before committing, confirming none of the excluded files appeared in the staged list.

```sh
git commit -m "Initial commit of Pontifex Zensical site"
```

---

## Step 5 — Creating the GitHub Repo and Pushing

A new **public** repo, `pontifex-zensical`, was created on GitHub — public was required, since GitHub Pages on a free plan only works with public repositories. No README, `.gitignore`, or license were initialized on GitHub's side, per GitHub's own guidance for pushing an existing repository.

```sh
git remote add origin https://github.com/calabiyau19/pontifex-zensical.git
```

```sh
git branch -M main
```

```sh
git push -u origin main
```

GitHub requires a personal access token in place of a password for this step. A fine-grained token was created, scoped to only this repository, with **Contents: Read and write** and **Workflows: Read and write** permissions (the latter is required specifically because Zensical's auto-generated `.github/workflows/docs.yml` lives under `.github/workflows/`).

---

## Step 6 — Enabling GitHub Pages and Fixing the First Build

On the new repo: **Settings → Pages → Source → GitHub Actions.**

The first Actions run failed immediately, before touching any content — because Pages hadn't been enabled yet, so there was nothing for the workflow to deploy to. Re-selecting "GitHub Actions" as the source resolved that.

The second run got further but failed at the actual build step:

```
Error: markdown-exec plugin is enabled, but markdown-exec is not installed.
```

Zensical doesn't bundle `markdown-exec` by default, even though the category index pages depend on it. The workflow's install step was updated:

```yaml
- run: pip install zensical markdown-exec
```

After committing and pushing that fix, the Actions build passed cleanly — a genuine build from scratch, on GitHub's own infrastructure, using only what was committed.

---

## Step 7 — Verifying at the Default URL First

Before touching `pontifex.site`, the site was confirmed working at its default GitHub Pages URL. This was the actual validation gate for the whole migration — proof the site builds and runs correctly outside of the local `zensical-vm` environment.

---

## Step 8 — The Domain Cutover

Since a custom domain can only be assigned to one repository at a time, the switch required two steps:

1. On `calabiyau19.github.io` → Settings → Pages → Custom domain → **Remove**
2. On `pontifex-zensical` → Settings → Pages → Custom domain → type `pontifex.site` → **Save**

The DNS check passed immediately (no propagation delay, since the underlying DNS records never changed — only which repo GitHub associates with the domain internally). Once GitHub finished provisioning a new TLS certificate for the domain under its new repo, **Enforce HTTPS** was re-enabled.

---

## How the Routing Actually Works

GitHub Pages' IP addresses are shared across every GitHub Pages site that exists — DNS alone doesn't identify which site you want. When a browser requests `pontifex.site`, it sends that hostname along with the request (via the `Host` header or TLS SNI). GitHub checks its internal registry — exactly what the "Custom domain" field in a repo's Pages settings writes to — to see which repository currently claims that domain, and serves that repo's content. GitHub enforces that only one repository can hold a given custom domain at a time, which is why the cutover required removing the domain from the old repo before adding it to the new one.

---

## Current State

- `pontifex.site` is live on Zensical, served from the `pontifex-zensical` repo, deployed automatically via GitHub Actions on every push
- HTTPS is enforced
- `calabiyau19.github.io` (the old Jekyll repo) still exists, still publicly reachable at its own default `.github.io` URL, but has no custom domain attached and is no longer connected to `pontifex.site`

---

## Remaining Items

- Remove the Jekyll line from `sync-websites-to-external.sh` on `lpt-HP`
- Decide, eventually, what to do with the unused `calabiyau19.github.io` repo — no urgency
- Card image light-mode fix, theme background hue experimentation (older deferred items)
