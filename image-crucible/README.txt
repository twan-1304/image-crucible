IMAGE CRUCIBLE V0.6.1 — INSTALLABLE/OFFLINE PACKAGE

Deploy every file in this folder together on an HTTPS website. Open the page once
while online, then use the “Install app” button when the browser offers it.

For local testing, serve this folder through localhost. Opening index.html directly
still runs the image tool, but browsers do not permit PWA installation from file://.

The app shell is cached after the first successful visit. Image processing, ZIP
inspection and conversion stay entirely on the device; no images are uploaded.

Supported ZIP entries: Stored and Deflate. Encrypted, multi-disk and ZIP64 archives
are rejected. Safe archives import immediately; Archive Forge keeps larger archives
inside a guarded, one-image-at-a-time memory budget.

V0.6.1 HOTFIX
- Downloads use an attached browser download link and hold the Blob URL long enough
  for Chrome/Edge, Firefox and Safari to begin saving it.
- Batch ZIP exports show an explicit “Save ZIP” action after the archive is built.
- Dragged or selected ZIPs automatically unpack compatible images into the queue
  after a successful preflight; unsafe or oversized archives remain blocked.
