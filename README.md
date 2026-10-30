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
- The original MyanArts parity targets are restored: TikTok phrase segments are up to 53 characters and YouTube phrase segments are up to 80 characters.
- Long cues are automatically split into readable upper/lower subtitle lines at a safe whitespace boundary; the example `ထိုရွာတွင် ဉာဏ်ကြီးသောသူတော်ကောင်းတစ်ဦး နေထိုင်ပါသည်။` becomes two lines without dropping text.
- Burmese sentence punctuation stays attached to the preceding phrase, so `သေးငယ်သောရွာလေးတစ်ရွာ ရှိပါသည်။` becomes `သေးငယ်သောရွာလေးတစ်ရွာ` / `ရှိပါသည်။`, never `ရှိပါသည်` / `။`.
- Burmese word boundaries are protected with `Intl.Segmenter("my", { granularity: "word" })`; words such as `အရက်` are never rendered as `အရ` / `က်`.
- Every generated result validates that the first subtitle starts at `00:00:00,000`, cues stay ordered, and the last SRT end equals the measured MP3 duration exactly.
- Cue order follows the ordered audio chunks.
- Chunk boundaries use measured audio durations. Where provider word-boundary metadata exists, those real boundaries are used; the Fast HTTP endpoint does not expose word boundaries, so its SRT uses the original MyanArts phrase segmentation with proportional timing rather than invented word timing.
- The final SRT cue is forced to the measured merged MP3 duration. The Server 1 path also scales its cue timeline to the measured merged duration before closing the final cue.

## Verification

```text
npm run build  PASS
npm test       PASS — 4 files, 17 tests
```

The old MyanArts UI was rebuilt on the published free link and checked in a real browser:

```text
Initial voice: ခွန် selected
Initial text: empty
Fast preset: Speed +30%, Pitch +5 Hz
SRT selector: TikTok / YouTube
ခွန် browser Generate: COMPLETE, MP3 + SRT controls visible
နန်း browser Generate: COMPLETE, MP3 + SRT controls visible
Two-line example live test: PASS — `ထိုရွာတွင် ဉာဏ်ကြီးသော` / `သူတော်ကောင်းတစ်ဦး နေထိုင်ပါသည်။`
Two-line test audio/SRT end: PASS — 2.976s / `00:00:02,976`
```

The EdgeTTS service live checks returned HTTP 200 `audio/mpeg` for both `my-MM-ThihaNeural` and `my-MM-NilarNeural`. SRT final timing is tied to measured merged audio duration.

## Limits

The input counter remains at 100,000 Unicode characters, but the free browser path has practical memory, sleep, rate-limit, and request-duration limits. The `Browser` button is intentionally listen-only and does not pretend to create an MP3. This recovery link does not restore the suspended `myanarts.onrender.com` Web/Worker or Auto Recap.
