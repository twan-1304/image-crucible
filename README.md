# Image Crucible installation recovery

This repository restores the original GitHub Pages launch address for existing
Image Crucible installations. Active application development lives in
https://github.com/twan-1304/remix.

## What it serves

- `/image-crucible/` and `/image-crucible/index.html` display the current ReMix
  workspace using the runtime files already hosted at `/remix/` on the same origin.
- The original `manifest.webmanifest` URL, including older query strings such as
  `?v=0.7.4`, remains available. It preserves the installed manifest identity
  `https://twan-1304.github.io/`, sets the launch URL and scope to `/remix/`, and
  supplies ReMix's name and icons.
- The page links a fresh manifest query to avoid an old cache entry while the
  retirement worker is updating. Chrome may ask the user to review the new name
  and icon. Browser update timing varies; the restored old launch page remains
  usable while an installation awaits its metadata update.
- `service-worker.js` stays at its original address. The page asks only the exact
  matching Image Crucible registration to update. The replacement worker deletes
  only `image-crucible-*` caches, unregisters itself, and has no fetch handler.
  It does not clear presets, browser storage, other app registrations or AI caches.
- No automatic page navigation or forced reload interrupts an open editor.

## Hosting

GitHub Pages: deploy from `main`, repository root. No build or backend is needed.
Keep this endpoint and the GitHub-hosted `/remix/` runtime available for returning
users, including during a later move to Cloudflare. A different origin requires a
separate migration; this compatibility site does not transfer installations to it.

## Repository rename consequence

The old repository name is now this compatibility project. Git remotes for the
main application must use `https://github.com/twan-1304/remix.git`.
This recovery site does not contain a second copy of the ReMix runtime or models.
