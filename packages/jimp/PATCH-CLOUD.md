# patch-cloud build of jimp@1.6.1

This tarball was built by patch-cloud from [fitzyracing1/jimp@patch-cloud](https://github.com/fitzyracing1/jimp/tree/patch-cloud),
a fork of [jimp-dev/jimp](https://github.com/jimp-dev/jimp). It is **not** an official upstream release.
Package name and version are unchanged from upstream (`jimp@1.6.1`, base `tag v1.6.1`).

## Fix

Ship the real Rollup browser bundle in dist/browser/index.js (and its index.d.ts) instead of the 98-byte stub published in jimp@1.6.1, so bundlers that resolve the "browser" export condition get Jimp, ResizeStrategy, etc.

- Upstream issue: https://github.com/jimp-dev/jimp/issues/1402

## Exactly what changed vs upstream

- turbo.json: build:browser now depends on the same package's build and declares dist/browser/** as an output, so tshy cannot overwrite the Rollup bundle and turbo cache hits restore it.
- Tarball: dist/browser/index.js (708 KB bundle) and dist/browser/index.d.ts (117 KB) replace the stubs; the shipped dist/browser/index.js.map is the same size as the one already published in 1.6.1. Other files match the published 1.6.1 tarball.
- Added this `PATCH-CLOUD.md` and a `_patchCloud` field in `package.json` (metadata only).

## Verification

- Reproduced the bug: with the original turbo.json, a cached rebuild produces the same 98-byte stub that was published; with the fix it produces the real bundle.
- Bundling import { Jimp, ResizeStrategy } from "jimp" with esbuild (platform=browser) yields defined exports, and the browser bundle creates, resizes and PNG-encodes an image; with upstream 1.6.1 esbuild fails with "No matching export in node_modules/jimp/dist/browser/index.js for import Jimp" (other bundlers silently yield undefined).

## Opting out

Remove the patch-cloud `registry=` override and reinstall from registry.npmjs.org to get the upstream tarball.

## GitHub Packages (scoped name)

The same patched build can be published to GitHub Packages as `@fitzyracing1/jimp@1.6.1`
(GitHub Packages only accepts names scoped to the repo owner). The unscoped name `jimp`
is what the patch-cloud registry serves; the scoped name is what GitHub Packages serves.

```ini
# .npmrc (project or ~/.npmrc). GitHub Packages npm needs a token even for public packages:
@fitzyracing1:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}   # a token with read:packages
```

```bash
# keep require('jimp') / import 'jimp' working via an npm alias:
npm install jimp@npm:@fitzyracing1/jimp@1.6.1
# or install under the scoped name:
npm install @fitzyracing1/jimp@1.6.1
```
