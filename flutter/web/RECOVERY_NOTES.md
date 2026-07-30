# Recovery notes (2026-07-30)

`flutter/web/` was deleted from upstream `rustdesk/rustdesk` on 2025-07-01
(commit `5faf0ad3cfef1b7237e6cbca53038d76ffdc281d`, "terminal works
basically. (#12189)") and was never restored, in upstream or in this fork.
This directory is a reconstruction, done as Phase 2 of
`rustdesk-api-web`'s `docs/WEBCLIENT_V2_REBUILD_PLAN.md` - see that doc for
the full context on why this is happening (short version: the webclient
bundle vendored in `rustdesk-api` embeds a real, compiled build of this
Dart app - `main.dart.js` - confirmed load-bearing against a live
deployment; the goal is to get a from-source build working again instead
of depending on an opaque vendored binary).

## Provenance

- Everything except `icons/` and `favicon.svg` was recovered from
  `5faf0ad3^` (the commit right before deletion), specifically its
  `flutter/web/v1/` subtree - flattened up to `flutter/web/` directly to
  match Flutter's expected web-platform layout (Flutter needs `web/index
  .html` at the platform-folder root, not nested under a `v1/` subdirectory
  - `android/`, `ios/`, `linux/`, `macos/`, `windows/` are all siblings of
    this folder for the same reason).
- `flutter/web/v2` (also present at `5faf0ad3^`) was **not** restored - its
  entire history is a single-line `README.md` reading "Under dev.". There
  was never anything to recover from it.
- `icons/` and `favicon.svg` were dropped in an earlier commit,
  `41a20b50ea04aab0e1c7cbe9f9fab3c0c5888d4b` ("split web js to v1 and v2",
  2024-06-22) - the same commit that reorganized the original `flutter/
  web/js/` into the `v1`/`v2` split. Restored from `41a20b50e^` (the commit
  right before that split), where they last existed at the same paths.
- `v1`'s own `js/` subdirectory is what the (currently broken, stale)
  upstream CI job in `.github/workflows/flutter-build.yml` (`web-basic`
  release job) still refers to as `flutter/web/js` - this restoration
  matches that expectation, so that workflow's build steps can be used as
  a reference for what still needs to happen here (`yarn install && yarn
  build` in `flutter/web/js`, `flutter build web --release` from `flutter/`).

## Known gap: this snapshot's own README said it was already stale

`flutter/web/README.md` (restored as-is, not edited) reads, in full:

> v1 is not compatible with current Flutter source code.

That's not a note added during this recovery - it's what the file already
said at `5faf0ad3^`, meaning the JS app was flagged as drifted from the
Dart side (`flutter/lib`) even before it was deleted. **Don't assume this
recovered snapshot builds cleanly against the current `flutter/lib` without
real reconciliation work** - this is exactly the kind of thing Phase 2's
exit criteria (a real connection through a from-source build) exists to
catch.

## What's still needed (not done, no Flutter toolchain available)

This recovery was done in a sandboxed environment with no Flutter/Dart
toolchain and no outbound access to package registries (npm, pub.dev) -
none of the following has been attempted or verified:

1. `flutter/web/js`'s codegen step (`ts_proto.py`/`gen_js_from_hbb.py`) -
   `js/src/message.ts`, `js/src/rendezvous.ts`, and `js/src/
   gen_js_from_hbb.ts` are gitignored (see `flutter/web/.gitignore`) and
   need to be generated before `yarn build` can run.
2. `yarn install && yarn build` in `flutter/web/js` actually succeeding
   (dependency resolution against `yarn.lock`, which is over a year old at
   this point - some pinned versions may no longer be resolvable).
3. `flutter build web --release` from `flutter/` actually succeeding
   against the restored `flutter/web/` scaffold and the current
   `flutter/lib` source, given the README's own staleness warning above.
4. Confirming the resulting `main.dart.js` can complete a real login
   handshake against `rustdesk-server`, and diffing its behavior against
   the currently-vendored bundle in `rustdesk-api/resources/web` rather
   than assuming they match.

This needs a real dev/CI environment with the Flutter toolchain - see
`rustdesk-api-web`'s plan doc, Phase 2 exit criteria and Risks section.
