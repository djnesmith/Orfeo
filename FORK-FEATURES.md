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

## 2. Closing the window quits on macOS

`window-all-closed` calls `app.quit()` on every platform. Upstream kept the app running on macOS with no window and no way to reopen it, since nothing handles the Dock's `activate`.

- Commits: the one that adds this entry (`git log -S "app.on('window-all-closed', () => app.quit())" --oneline`)
- Files: `electron/main.ts`
- Check: open Orfeo, click the red close button; the app leaves the Dock and `pgrep -f Orfeo.app` prints nothing.

## 3. macOS menu bar

On macOS the app gets Electron's standard app menu (About, Hide, Quit ⌘Q) and Edit menu (Undo, Redo, Cut, Copy, Paste, Select All). Upstream sets no menu at all, so ⌘Q and ⌘C/⌘V/⌘X/⌘A did nothing. Windows still gets `null`. Neither role adds zoom shortcuts, so the app's own ⌘+/⌘- zoom stays the only one.

- Commits: the one that adds this entry (`git log -S "role: 'editMenu'" --oneline`)
- Files: `electron/main.ts`
- Check: the menu bar shows Orfeo and Edit. Open Settings (Ctrl+,), type in "Search settings…", ⌘A then ⌘X empties it, ⌘V pastes it back. ⌘Q quits.

## 4. Mac icon

`public/icon.icns`, which upstream's `package.json` already names as `mac.icon` but never added. It is the Orfeo mark from `src/components/OrfeoMark.tsx` in `#E68621` on transparent, sized and placed like `public/icon.ico` (within 0.4% RMSE at 256px), rendered from the SVG path so it stays sharp at 1024px.

- Commits: the one that adds this entry (`git log --oneline -- public/icon.icns`)
- Files: `public/icon.icns`
- To remake it: wrap the `d="…"` path from `OrfeoMark.tsx` in a 1024×1024 SVG as `<g transform="translate(138.5 92) scale(5.6)"><path fill="#E68621" d="…"/></g>`, render 16–1024px PNGs into `Orfeo.iconset` with `rsvg-convert` (names `icon_<n>x<n>.png` and `@2x`), then `iconutil -c icns Orfeo.iconset -o public/icon.icns`.
- Check: after a build, the Dock shows the orange Orfeo mark, not the Electron atom.
