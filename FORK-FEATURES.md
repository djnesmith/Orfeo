# Fork features

Read this before merging upstream. Every item must still work after the merge.

Branch: `main`. Base: `upstream/main` (Squarebow/Orfeo), forked at `6e7dd5e` (tag `v1.6.0`).

Upstream only ships Windows builds. On a Mac the native audio engine (`native/`, WASAPI + FluidSynth DLLs) is skipped and the app uses its built-in JavaScript synth.

Typecheck: `npm run typecheck`. Build a Release Mac app (Apple Silicon):

```sh
npm ci
npm run build
npx electron-builder --mac dir --arm64 --publish never -c.mac.identity=null
codesign --force --deep -s - release/mac-arm64/Orfeo.app
ditto release/mac-arm64/Orfeo.app /Applications/Orfeo.app
```

## Features

## 1. Hand-engine check ignores line endings

`scripts/check-hand-engine-version.mjs` turns CRLF into LF before hashing `src/utils/handAssignment.ts`. Upstream's stored hash came from a Windows (CRLF) checkout, so `npm run build` failed on every Mac/Linux (LF) checkout. `src/utils/handAssignment.hash.json` now holds the LF hash.

- Commits: the one that adds this entry (`git log -S 'Line endings ignored' --oneline`)
- Files: `scripts/check-hand-engine-version.mjs`, `src/utils/handAssignment.hash.json`
- After a merge: upstream rewrites `handAssignment.hash.json` on each engine bump with a CRLF hash. Keep their `version`, then run `node scripts/check-hand-engine-version.mjs --update` to store the LF hash.
- Check: `npm run check:hand-engine` exits 0.
