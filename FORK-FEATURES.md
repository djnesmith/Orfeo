# Fork features

Read this before merging upstream. Every item must still work after the merge.

Branch: `main`. Base: `upstream/main` (Squarebow/Orfeo), forked at `6e7dd5e` (tag `v1.6.0`). Last synced with upstream `880bb7c` (v1.7.3).

As of v1.7.0 upstream builds macOS itself: `npm run dist:mac` makes an ad-hoc signed dmg and zip for Apple Silicon and Intel. Its native audio engine still runs on Windows only. `scripts/build-native.mjs` can now compile it on macOS (FluidSynth from source, 182e024), but `connectAudioHost` in `electron/main.ts` refuses it on any non-Windows system, and the Mac package ships no `native/` files. On a Mac the app uses its built-in JavaScript synth.

Typecheck: `npm run typecheck`. Tests: `npm test`. Build a Release Mac app (Apple Silicon):

```sh
npm ci
npm run dist:mac -- --publish never   # release/mac-arm64/ and release/mac/ (Intel), plus dmg and zip; --arm64 is ignored
ditto release/mac-arm64/Orfeo.app /Applications/Orfeo.app
```

## Features

Orfeo now follows upstream's Mac support (David, 2026-11-07): where upstream has its own version of a Mac change, take upstream's code as is, and don't port ours on top. There are no fork-only features right now. The four below are superseded by upstream; they stay listed so the next merge can check upstream kept them.

## 1. Hand-engine check ignores line endings — now in upstream (9c3dd44)

`scripts/check-hand-engine-version.mjs` turns CRLF into LF before hashing `src/utils/handAssignment.ts`, and upstream's `handAssignment.hash.json` now holds the LF hash. The v1.7.0 merge took upstream's version of the script.

- Check: `npm run check:hand-engine` exits 0.

## 2. Closing the window quits on macOS — now in upstream (d01b02e)

`window-all-closed` calls `app.quit()` on every platform. The v1.7.0 merge took upstream's line.

- Check: open Orfeo, click the red close button; the app leaves the Dock and `pgrep -f Orfeo.app` prints nothing.

## 3. macOS menu bar — now in upstream (d01b02e)

On macOS upstream builds the app, Edit and Window menus (ours had app and Edit only). Windows still gets `null`. None of these roles adds page-zoom shortcuts, so the app's own ⌘+/⌘- zoom stays the only one. The v1.7.0 merge took upstream's code.

- Check: the menu bar shows Orfeo, Edit and Window. Open Settings (Ctrl+,), type in "Search settings…", ⌘A then ⌘X empties it, ⌘V pastes it back. ⌘Q quits.

## 4. Mac icon — now in upstream (a48555d)

Upstream added `build/icon.icns` (the orange Orfeo mark on a dark rounded square) and points `mac.icon` at it. Our `public/icon.icns` (the bare orange mark on transparent) was no longer referenced, so the v1.7.0 merge removed it. It is still in git at `fda14d2` if wanted back.

- Check: after a build, the Dock shows the Orfeo mark, not the Electron atom.
