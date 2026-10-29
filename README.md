# MyanArts Free TTS Studio

A static, browser-first Myanmar TTS utility. Thiha is selected by default and Nilar is available as an alternative. The original MyanArts Fast Direct aliases are also restored: `ကျည်ကွန်ဒို · ခွန်` maps to `my-MM-ThihaNeural`, and `ကျည်ကွန်ဒို · နန်း` maps to `my-MM-NilarNeural`. The two Fast voices call the replacement EdgeTTS service deployed in the active `hkunzeya@gmail.com` Render workspace; the original Server 1 route remains available for Thiha/Nilar.

## What is implemented

- Thiha default (`my-MM-ThihaNeural`) and Nilar optional (`my-MM-NilarNeural`) through Server 1 with the user-entered PIN.
- Original Fast aliases restored: ခွန် (male) and နန်း (female), both under the ကျည်ကွန်ဒို label, through the live EdgeTTS service at `https://myanarts-edgetts-95d9.onrender.com`.
- Speed and pitch controls. Fast voices select Speed `+30%` and Pitch `+10 Hz`; the EdgeTTS API receives the existing speed value and a documented 10 Hz ≈ 1 semitone pitch mapping.
- Fast voices do not require the Server 1 PIN. The Server 1 PIN field is disabled for them.
- Direct browser CORS request to EdgeTTS: ordered 3,000-character chunks, bounded concurrency, retry of the failed chunk only, MP3 Blob assembly, measured duration, and SRT download.
- 3,000-character application chunks with exact Unicode reconstruction checks.
- Actual browser audio metadata measurement for every chunk and the merged MP3; the final SRT cue is forced to the measured final duration. The EdgeTTS API does not expose word boundaries, so Fast-voice SRT uses measured chunk-level cues rather than invented word timing.
- 100,000-character input counter with a practical 40-chunk per-run limit and honest workload warnings.
- Native browser Listen button remains a listen-only fallback and is not presented as an MP3 generator.
- Responsive dark Myanmar audio-studio UI.

## Verification

```text
npm run build  PASS
npm test       PASS — 4 files, 10 tests
```

Live EdgeTTS verification on 2026-10-30:

```text
GET /health                  200
OPTIONS /api/speech          204 with configured CORS origin
ခွန် my-MM-ThihaNeural       200 audio/mpeg, 16,992 bytes, ffprobe 2.832 sec
နန်း my-MM-NilarNeural       200 audio/mpeg, 19,296 bytes, ffprobe 3.216 sec
```

The EdgeTTS service is a self-hosted wrapper around Microsoft's online Edge TTS transport. It may sleep on the free plan, rate-limit, or change if Microsoft changes the upstream service. The old browser-only Microsoft WebSocket path remains research-only because its real browser smoke test failed with close code `1006`.

## Local preview

```bash
npm install
npm run dev -- --host 0.0.0.0 --port 4173
```

## Privacy and availability

No Server 1 PIN or API key is embedded in this site. The Server 1 PIN is held in browser memory only and is not persisted. The replacement EdgeTTS endpoint is intentionally configured for this free studio's browser origin with rate limiting and no embedded API key; it is a public user-facing service and should not be used as an unrestricted general API. This path does not restore the suspended `myanarts.onrender.com` workspace, Auto Recap, or the background worker.
