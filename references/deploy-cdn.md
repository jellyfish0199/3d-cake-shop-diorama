# Static deployment & CDN cache rule

## Local verification

```bash
cd <project-root>
node build-static.mjs          # builds dist/ (~32 runtime resources)
python3 -m http.server 8765    # serve the project root
# headless Chrome + Playwright (SwiftShader) for screenshots/QA
```

Opening `dist/index.html` via `file://` will NOT work — ES modules require
http(s).

## Studio publish

```bash
node /mods/neta/neta work publish web works/webs/sugar-dreamland-cake-shop \
  --from works/webs/sugar-dreamland-cake-shop/dist
```

The publish output's `preview.url` (neta.art shell) embeds a content hash, so
each publish is cache-fresh.

## Public share CDN — the cache trap

The `public.cohub.run/s/<spaceId>/...` share host caches **per path**, ignores
query strings (`?v=1` does nothing), and overwritten files keep serving their
old cached copy. Rules:

1. Never point users at an already-shared path after changing content.
2. Copy `dist/` to a NEW versioned directory each release:
   `sugar-dreamland-cake-shop-v11`, `-v12`, …
3. `mkdir -p` the new dir, `cp -r dist/. <newdir>/`, wait ~5s, then `curl`
   the new path and grep for the change you just made before handing out the
   link.
4. Old paths cannot be reliably purged; treat them as frozen snapshots.

## Generic static hosts (GitHub Pages / Netlify / any CDN)

- The `dist/` output is directly deployable: `index.html` at the root, all
  references relative, so sub-path hosting (e.g. `user.github.io/repo/`) works
  without configuration.
- GitHub Pages: upload the built directory (or build in CI), Settings → Pages
  → deploy from branch root.
- Keep the `<hash>`-style cache busting in mind for other CDNs too: prefer
  versioned paths over query strings when a CDN ignores queries.
