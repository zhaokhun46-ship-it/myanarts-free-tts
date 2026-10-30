# MyanArts Free TTS Studio

A browser-first free TTS recovery page that keeps the original MyanArts TTS Studio interaction pattern: Myanmar Text Input, grouped voice cards, Speed/Pitch/Volume Booster controls, Output Filename, Generate AI Speech, AI preview, Browser listen, generated audio, MP3/SRT downloads, and measured timing status.

## Current defaults

- Default voice: `ခွန်` under `ကျည်ကွန်ဒို`.
- Text input: intentionally blank on first load so the user can enter their own script.
- Default Fast settings: Speed `+30%`, Pitch `+5 Hz`.
- SRT type: TikTok by default; YouTube is selectable.

## Voice routing

- `Thiha` and `Nilar` retain the original Azure AI / Server 1 route and ask for a PIN only at generation time. The PIN stays in browser memory and is never persisted.
- The original Fast Direct aliases are restored in the old UI under `ကျည်ကွန်ဒို`: `ခွန်` maps to `my-MM-ThihaNeural`, and `နန်း` maps to `my-MM-NilarNeural`.
- `ခွန်` and `နန်း` use the active replacement EdgeTTS service at `https://myanarts-edgetts-95d9.onrender.com`, require no Server 1 PIN, and select Speed `+30%` / Pitch `+5 Hz` by default.
- Generate requests are local-browser initiated. Text is split into ordered chunks, failed chunks retry independently, and the final MP3/SRT stays in the browser.

## SRT verification

- SRT cues preserve the complete source text and split at Myanmar sentence punctuation/whitespace boundaries.
- Subtitle cues are shorter: up to 44 characters for TikTok and 60 for YouTube. Rendered text rows target at most 22 characters for TikTok and 30 for YouTube.
- Long cues are automatically split into readable lines at whitespace first, then safe Myanmar word boundaries. A single indivisible word is kept intact rather than split mid-word.
- Burmese sentence punctuation stays attached to the preceding phrase, so `သေးငယ်သောရွာလေးတစ်ရွာ ရှိပါသည်။` becomes `သေးငယ်သောရွာလေးတစ်ရွာ` / `ရှိပါသည်။`, never `ရှိပါသည်` / `။`.
- Burmese word boundaries are protected with `Intl.Segmenter("my", { granularity: "word" })`; words such as `အရက်` are never rendered as `အရ` / `က်`.
- Every generated result validates ordered cue times and that the last SRT end equals the measured MP3 duration exactly. A provider-reported first-word offset is retained when the audio contains leading silence.
- The v1 `AI မြန်မာ` group is restored with six Server 1 voices: Elias, Qianluo, Dama, Sofia, Muyao, and Faisal. They keep the v1 `+10%` speed / `0Hz` pitch defaults and require the user-entered Server 1 PIN in browser memory only.
- Both `ကျည်ကွမ်ဒို` (EdgeTTS service) and `Azure AI` / `AI မြန်မာ` (Server 1 web route) call the same subtitle formatter and exact-duration timeline validation.
- Cue order follows the ordered audio chunks.
- `ကျည်ကွန်ဒို` calls the timed EdgeTTS endpoint, which returns Microsoft Edge `WordBoundary` events with the MP3. When the provider's boundary text aligns with the source, SRT cue starts and intermediate ends use those provider offsets. If boundary text differs, the valid MP3 is retained and SRT is rebuilt from the exact source text with estimated phrase timing; the UI warns that line-level sync should be reviewed.
- Server 1 provider SRT cue timestamps are preserved when chunks are placed on the merged timeline. The implementation no longer stretches every cue across the whole file; it clamps out-of-range cues and sets only the final cue end to the measured merged MP3 duration.

## Verification

```text
npm run build  PASS
npm test       PASS — 6 files, 33 tests

Live browser test of compact TikTok lines on release `e80ead9`: short Khun MP3 generated successfully, the two-line Burmese example rendered correctly, and the SRT final timestamp exactly matched the measured audio duration. Existing real-generation checks confirmed Azure AI Thiha also produces MP3/SRT with source text preserved.
```

The old MyanArts UI was rebuilt on the published free link and checked in a real browser:

```text
Initial voice: ခွန် selected
Initial text: empty
Fast preset: Speed +30%, Pitch +5 Hz
SRT selector: TikTok / YouTube
ခွန် browser Generate: COMPLETE, MP3 + SRT controls visible
နန်း browser Generate: COMPLETE, MP3 + SRT controls visible
Compact-line live test: PASS — `သေးငယ်သောရွာလေးတစ်ရွာ` / `ရှိပါသည်။` (21 and 9 characters)
Compact-line test audio/SRT end: PASS — 2.184s / `00:00:02,184`
```

The deployed EdgeTTS service commit `f058db8` returned HTTP 200 from `/api/speech-with-timing`, including seven ordered Thiha word-boundary events and valid MP3 bytes. The live response included CORS permission for this static Studio origin. The sample's measured MP3 duration was `6,240 ms`; the last provider word ended at `5,608 ms` (the final SRT caption still closes at the measured audio end to keep the prior exact-duration contract).

The latest live Studio run measured the Khun MP3 at `7,416 ms`; SRT also ended at `7,416 ms`, with the source text preserved and `အရက်` intact. The Azure AI Thiha run measured `6,606 ms`; SRT ended at `6,606 ms`, restored sentence punctuation omitted from provider cue text, and preserved the complete source wording. Both audio elements loaded to `readyState=4` in the browser.

The latest user-reported failure was traced to client-side SRT source-alignment validation after successful EdgeTTS requests, not an audio-provider outage. Boundary normalization now ignores punctuation/whitespace only while matching and restores the original input text. A genuine lexical mismatch keeps the MP3 and produces an exact-source SRT with estimated phrase timing plus an explicit warning instead of failing the entire generation.

## Limits

The input counter accepts 100,000 Unicode characters. Paste handling is explicit rather than relying on native textarea truncation, so a full Myanmar clipboard payload is inserted and counted exactly (`100,000 / 100,000` verified in the browser). The free browser path still has practical memory, sleep, rate-limit, and request-duration limits. The `Browser` button is intentionally listen-only and does not pretend to create an MP3. This recovery link does not restore the suspended `myanarts.onrender.com` Web/Worker or Auto Recap.
