# Fork features

Read this before merging upstream. Every item must still work after the merge.

Branch: `main`. Base: `upstream/main` (Squarebow/Orfeo), forked at `6e7dd5e` (tag `v1.6.0`).

Upstream only ships Windows builds. On a Mac the native audio engine (`native/`, WASAPI + FluidSynth DLLs) is skipped and the app uses its built-in JavaScript synth.

Typecheck: `npm run typecheck`. Build a Release Mac app (Apple Silicon):

```sh
npm ci
npm run copyworklet
npx electron-vite build
npx electron-builder --mac dir --arm64 --publish never -c.mac.identity=null -c.mac.icon=null
codesign --force --deep -s - release/mac-arm64/Orfeo.app
ditto release/mac-arm64/Orfeo.app /Applications/Orfeo.app
```

`npm run build` and `npm run dist` fail on a Mac at `check:hand-engine`: `src/utils/handAssignment.hash.json` holds the hash of a Windows (CRLF) checkout, so an LF checkout never matches. The steps above skip that check. `package.json` points `mac.icon` at `public/icon.icns`, which upstream doesn't have, so the app gets the default Electron icon.

## Features

None yet.
