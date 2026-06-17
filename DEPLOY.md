<!--
SPDX-FileCopyrightText: 2025-2026 SKY, LLC.
SPDX-License-Identifier: MPL-2.0
-->
# Deploying uffs.io

This repo **is** the site: a single self-contained `index.html` (inline CSS/JS,
no build framework) plus `assets/`, served by **GitHub Pages**.

## Hosting at a glance

| Thing | Value |
|---|---|
| Pages build type | **GitHub Actions** (`.github/workflows/pages.yml`), not "deploy from a branch" |
| Production URL | <https://uffs.io/> |
| Custom domain | `uffs.io` — set by the `CNAME` file **and** the Pages `cname` setting |
| `skyllc-ai.github.io` | **301-redirects to `uffs.io`** (automatic + non-disableable while a custom domain is set) |
| TLS | GitHub-managed; re-provisions automatically after any domain change (can take a few minutes) |

A push to `main` triggers the workflow, which **stamps the footer**, then deploys.

## The footer build stamp (auto, drift-free)

The footer shows `Site build · <date> v<N> (<short-sha>)`, e.g.
`Site build · 2026-06-17 v5 (b4e7e27)`:

- `<date>` — the deploy commit's date.
- `v<N>` — the **Nth commit that calendar day** (same-day edits read `v1`, `v2`, …).
- `<short-sha>` — exact commit, so you can match the live page to the repo.

In source, `index.html` carries the literal `Site build · dev`; the workflow's
`sed` replaces it at deploy time. **If the live site ever shows `… · dev`, the
deploy workflow didn't run** (check the Actions tab).

## Toggling the custom domain

`uffs.io` (custom) ⟺ `skyllc-ai.github.io` (direct) is an either/or — a custom
domain *always* makes the `*.github.io` host redirect, so you can't have both
serve directly on one Pages site.

**Disable `uffs.io` (serve directly at `skyllc-ai.github.io`):**
```bash
git rm CNAME && git commit -m "ci: drop custom domain" && git push
gh api -X PUT /repos/skyllc-ai/skyllc-ai.github.io/pages -f cname=''
```

**Re-enable `uffs.io`:**
```bash
echo uffs.io > CNAME && git add CNAME && git commit -m "ci: restore custom domain" && git push
gh api -X PUT /repos/skyllc-ai/skyllc-ai.github.io/pages -f cname=uffs.io
```
(The TLS cert re-provisions on its own afterward.)

## Gotcha: a new domain looks "blocked" on some networks

A **newly-registered domain** is often filtered for ~24–48 h by security DNS
resolvers (Cisco **Umbrella** / OpenDNS, NextDNS "newly-seen domains", many
corporate filters). Symptom: `uffs.io` resolves to an Umbrella **block-page IP**
(`146.112.61.x`) instead of GitHub Pages (`185.199.108–111.153`), so the page
"won't load" — but **only on that network**. GitHub and the public DNS are fine.

Diagnose (compare your resolver vs. the truth):
```bash
dig +short uffs.io A                 # your resolver — 146.112.61.x ⇒ Umbrella block
dig +short @1.1.1.1 uffs.io A        # public — should be 185.199.108–111.153
# Prove GitHub serves it, bypassing the block:
curl -sSI --resolve uffs.io:443:185.199.108.153 https://uffs.io/ | head -1   # → HTTP/2 200
```

Fix (pick one): **allowlist `uffs.io` in the Umbrella/OpenDNS dashboard**
(network-wide, best), add a local `hosts` override, or just wait it out.
**Do not** drop the custom domain to "fix" this — that only breaks `uffs.io`
for everyone else, who can reach it fine.

## Verify a deploy

```bash
gh run list --repo skyllc-ai/skyllc-ai.github.io --workflow=pages.yml --limit 3
curl -sSI https://skyllc-ai.github.io/ | grep -i location   # → https://uffs.io/
curl -s  https://uffs.io/ | grep -o 'Site build · [^<]*'    # the live stamp
```
