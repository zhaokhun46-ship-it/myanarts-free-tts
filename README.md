# MyanArts Free TTS Studio

A browser-first free TTS recovery page that keeps the original MyanArts TTS Studio interaction pattern: Myanmar Text Input, grouped voice cards, Speed/Pitch/Volume Booster controls, Output Filename, Generate AI Speech, AI preview, Browser listen, generated audio, MP3/SRT downloads, and measured timing status.

## Voice routing

- `Thiha` (default) and `Nilar` retain the original Azure AI / Server 1 route and ask for a PIN only at generation time. The PIN stays in browser memory and is never persisted.
- The original Fast Direct aliases are restored in the old UI under `ကျည်ကွမ်ဒို`: `ခွန်` maps to `my-MM-ThihaNeural`, and `နန်း` maps to `my-MM-NilarNeural`.
- `ခွန်` and `နန်း` use the active replacement EdgeTTS service at `https://myanarts-edgetts-95d9.onrender.com`, require no Server 1 PIN, and select Speed `+30%` / Pitch `+10 Hz` by default.
- Generate requests are local-browser initiated. Text is split into ordered chunks, failed chunks retry independently, and the final MP3/SRT stays in the browser.

## Verification

```text
npm run build  PASS
npm test       PASS — 4 files, 10 tests
```

The old MyanArts UI was rebuilt on the published free link and checked in a real browser:

```text
Default UI voice: Thiha
Fast card: ကျည်ကွန်ဒို · ခွန် / နန်း
Fast preset: Speed +30%, Pitch +10 Hz
Fast PIN field: not required
ခွန် browser Generate: COMPLETE, MP3 + SRT controls visible
နန်း browser Generate: COMPLETE, MP3 + SRT controls visible
```

The EdgeTTS service live checks returned HTTP 200 `audio/mpeg` for both `my-MM-ThihaNeural` and `my-MM-NilarNeural`. SRT final timing is tied to measured merged audio duration. Fast-voice SRT uses measured chunk-level cues because the HTTP service does not expose word boundaries.

## Limits

The input counter remains at 100,000 Unicode characters, but the free browser path has practical memory, sleep, rate-limit, and request-duration limits. The `Browser` button is intentionally listen-only and does not pretend to create an MP3. This recovery link does not restore the suspended `myanarts.onrender.com` Web/Worker or Auto Recap.
