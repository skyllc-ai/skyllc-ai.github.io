# skyllc-ai.github.io

Canonical landing page for **UFFS — UltraFastFileSearch**, served by GitHub Pages at:

> **https://uffs.io/**  (`https://skyllc-ai.github.io/` redirects here)

This is the org user-page repo. Any file at the repo root is served from the site root, so
`index.html` is the homepage. `.nojekyll` tells Pages to serve files verbatim (no Jekyll build).

> **Deploying, the `uffs.io` domain, the footer build stamp, and the "new domain looks
> blocked" (Umbrella) gotcha → [DEPLOY.md](DEPLOY.md).**

## Structure

```
index.html              Flagship benchmark landing page (self-contained, no build step)
robots.txt / sitemap.xml SEO basics
assets/brand/           Wordmark, hero image, favicons (mirrored from the UFFS repo)
assets/charts/          Benchmark SVGs (mirrored from UFFS docs/benchmarks/charts/)
```

## Editing

`index.html` is plain HTML with an embedded `<style>` block — no toolchain, no dependencies.
Edit and push; GitHub Pages redeploys within ~1 minute.

To refresh benchmark numbers or charts, copy the latest SVGs from the UFFS repo:

```bash
cp ../UltraFastFileSearch/docs/benchmarks/charts/<version>/*.svg assets/charts/
```

…and update the version-stamped figures in the proof strip / captions in `index.html`.

## Enabling Pages (one-time)

Repo **Settings → Pages → Build and deployment → Source: Deploy from a branch**,
branch `main` / folder `/ (root)`. The site goes live at `https://skyllc-ai.github.io/`.

## Moving to a custom domain later (e.g. skyllc.ai)

1. Buy the domain.
2. Add a `CNAME` file at this repo root containing only the bare domain (e.g. `skyllc.ai`).
3. DNS: add the GitHub Pages `A`/`AAAA` apex records (or a `CNAME` for `www`) per
   <https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site>.
4. Settings → Pages → Custom domain → enter the domain, enable **Enforce HTTPS**.
5. Update `<link rel="canonical">`, `og:url`, `twitter:image`, and `sitemap.xml` in `index.html`
   to the new domain.

GitHub Pages automatically **301-redirects** the old `*.github.io` URLs to the custom domain,
so links already shared on HN / Reddit / LinkedIn keep working.
