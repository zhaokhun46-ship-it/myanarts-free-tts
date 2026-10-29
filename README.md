# MyanArts Free TTS Studio

A static, browser-first Myanmar TTS utility. Thiha is selected by default and Nilar is available as an alternative. The app keeps the text local, sends generation requests directly from the browser to the existing Server 1 TTS endpoint, and downloads ordered MP3 + Myanmar SRT files without depending on the suspended MyanArts Render Web/Worker.

## What is implemented

- Thiha default (`my-MM-ThihaNeural`) and Nilar optional (`my-MM-NilarNeural`).
- Speed and pitch controls.
- Server 1 PIN is entered by the user at runtime, held in browser memory only, never written into source, local storage, or the repository.
- Direct browser CORS request to Server 1: login, chunk submission, bounded parallel jobs, ordered result collection, retry of failed chunks, MP3 download, and SRT download.
- 3,000-character application chunks with exact Unicode reconstruction checks.
- Actual browser audio metadata measurement for each chunk and the merged MP3; the final SRT cue is forced to the measured final duration.
- 100,000-character input counter with a practical 40-chunk per-run limit and honest workload warnings.
- Native browser Listen button remains a listen-only fallback and is not presented as an MP3 generator.
- Responsive dark Myanmar audio-studio UI.

## Verification

```text
npm run build  PASS
npm test       PASS — 3 files, 8 tests
```

The original browser-only Microsoft Edge WebSocket path was smoke-tested in the sandbox and failed before `onopen` with close code `1006`; it remains in the source as research/fallback code but is not claimed as the reliable generation route. The recovery path uses the live Server 1 endpoint instead. The external endpoint can still rate-limit, sleep, or change its access policy.

## Local preview

```bash
npm install
npm run dev -- --host 0.0.0.0 --port 4173
```

## Privacy and availability

No API key is embedded in this site. The Server 1 PIN must be entered by the user when generating audio and is not persisted by this app. Server 1 is an external service; this static page does not restore the suspended `myanarts.onrender.com` workspace and does not restore Auto Recap or the background worker.
