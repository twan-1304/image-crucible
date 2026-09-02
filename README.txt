# Image Crucible V0.7.1

Image Crucible handles images and videos entirely in the browser.

## V0.7.1

- Video ETA samples are tracked separately for native WebM and FFmpeg encoding, preventing fast WebM timings from producing absurd estimates for long 4K MP4/MOV jobs.

## Video Forge

- Imports MP4, MOV, WebM, MKV, AVI, MPEG, MTS/M2TS, 3GP and OGV when the browser can read their metadata.
- Exports MP4, MOV and WebM.
- H.264 encoding for MP4/MOV, VP8 and VP9 for WebM.
- Visual-quality (CRF) and explicit bitrate modes.
- Maximum resolution, frame-rate, encoding-speed and audio controls.
- Non-destructive start/end trimming and multiple internal cuts.
- Direct folder saving, individual downloads, batch ZIP export and CSV reports.

### Which output to pick

**MP4 / H.264 is the recommended path.** It runs on the FFmpeg WebAssembly engine inside a module Worker, so it is unaffected by tab focus, window visibility or screen sleep. Conversion is slower than real time - the engine is a single-threaded build - but it finishes unattended.

**WebM / VP8 or VP9 is real-time only.** It bypasses ffmpeg.wasm's libvpx encoders entirely: Chrome or Edge records the resized canvas and captured source audio through its native VP8/Opus or VP9/Opus MediaRecorder. Encoding runs at roughly 1x, so a 30-second clip takes about 30 seconds, but the pipeline plays the source through a hidden video element and captures the canvas, which makes it sensitive to anything that interrupts playback:

- The tab must stay visible. Chrome clamps background timers to one per second, which would otherwise reduce the output to a slideshow. Recording pauses automatically when the tab is hidden and resumes when it returns, so the result stays correct - the job simply takes longer.
- A screen wake lock is requested for the duration of the recording where the browser supports it.
- If the effective frame rate still falls below 60% of the target, the job completes but reports the measured rate as a warning. Check that output before sending it out.
- Keeping audio routes the source through Web Audio, which drives the playback clock. The graph is built after the source has loaded, and a suspended audio context is reported as a named error rather than a silent freeze. Selecting **Remove audio** takes Web Audio out of the path entirely.

VP9 quality mode targets roughly 28% less bitrate than VP8 for comparable output. If results look softer than expected, use bitrate mode instead.

### Known limitations

- VP9 through ffmpeg.wasm is not available. The `libvpx-vp9` encoder is present in `@ffmpeg/core@0.12.10` but traps with `RuntimeError: memory access out of bounds` on the first frame, including with `-lag-in-frames 0 -auto-alt-ref 0`, which rules out lookahead-buffer exhaustion. The encoder branch remains in the source, dormant, pending an upstream fix. Until then, background-safe encoding means MP4/H.264.
- Import is gated on the browser decoding the source. Containers Chrome cannot read natively - MKV, AVI, MPEG-TS/MTS, ProRes MOV, and often H.265 in MP4 - fail as **Unreadable video** before any encoding starts, and "Convert all" skips failed jobs. Remux to MP4/H.264 first.
- WebM files produced by the native recorder carry Chrome's `alpha_mode=1` track tag despite being 4:2:0 without alpha. Some editors and older players treat this as an alpha channel. Prefer MP4 for footage headed into an NLE.
- 4K sources are slow. The FFmpeg engine is single-threaded, so encode time scales with output pixels: expect roughly 10-15x real time for a 2160p output and 4-6x for a 1080p output. Cap max resolution at 1920x1080 unless you specifically need 4K out.
- The browser build uses a 1.5 GB per-video safety ceiling. Source files are mounted read-only through WORKERFS rather than copied into WebAssembly memory, so only the output and the encoder's working set occupy the heap; browsers that reject the mount fall back to loading the whole source into memory.

MP4 and MOV download the FFmpeg WebAssembly engine from jsDelivr on their first conversion and cache it in the service worker when served over HTTP. Until that first successful download completes, the MP4 path needs network access. The standalone HTML also works when opened directly through `file://`, where no service worker registers and nothing is cached between runs; its small worker modules are fetched and assembled into one local Blob worker so browsers never need to construct a cross-origin Worker. The module Worker loads the matching ESM core build. WASM memory crashes automatically discard the damaged FFmpeg engine before the next MP4/MOV conversion. Source media is never uploaded.

## Planned

WebCodecs (`VideoEncoder` / `VideoDecoder`) as the primary engine, with ffmpeg.wasm retained as a fallback demuxer for containers the browser cannot read. That removes the CDN dependency, lifts the memory ceiling, uses hardware encoding where available, and makes background-safe VP9 possible.

## Deploy

Publish these runtime files together:

- `index.html`
- `service-worker.js`
- `manifest.webmanifest`
- `icons/icon.svg`
- `icons/icon-192.png`
- `icons/icon-512.png`
- `THIRD_PARTY_NOTICES.md`

Serve them over HTTPS or localhost so the PWA and direct folder export features can work.
