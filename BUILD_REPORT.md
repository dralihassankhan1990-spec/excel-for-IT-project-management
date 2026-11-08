# Batch 1 v5 rebuild report — demos 01–11 (Gemini voice)

Date: 2026-11-02. Coordinator: Batch 1 subagent. Status: STOPPED EARLY on
free-tier quota. 1 of 11 demos built and verified. 10 not built. Nothing was
uploaded, moved, or deleted. `output/demos/`, `output/demos_v4/`, all backup
folders, and Drive were not touched.

## Voice (locked, used for every chunk)

- Model: `gemini-3.8-flash-tts`. Voice: `Achird`. Style, exactly:
  "warm, friendly and clear, like a patient teacher explaining step by step,
  calm steady pace".
- Audio ONLY through the connector helper
  `~/workspace/custom-connectors/custom.google-gemini/bin/gemini_tts.py`
  (`synth`, `--text-file`). No API key appears in any file, argument, or log.
- Chunking: 5 chunks per demo, same chunking as the v4 builders. Segment
  video durations equal the measured Gemini chunk MP3 durations, so picture
  changes land on their sentences.
- Chunk texts: v4 builder texts verbatim. NO Helio pronunciation map was
  applied (no "SUM IF", "COUNT IFS", "E-DATE", etc.). The one planned
  restoration, demo 08 chunk 3 "I T zero zero paanch" -> "IT-005", was NOT
  reached because the batch stopped at the quota before demo 08. It must
  still be applied on resume.
- The two pre-generated full-script reference files in
  `voice_samples/google_tts/` were NOT used as demo audio and were not
  modified.

## Builder

- `~/workspace/excel-training/build_v5_batch1.py`. It imports
  `build_batch_A.py` (demos 01–04, 08–11) and `build_demos_v4.py`
  (demos 05–07) and monkeypatches only their synth functions and OUT/WORK
  paths, so visuals are byte-for-byte the v4 rendering code.
- VISUAL NOTE for the parent: demos 05–07 rendered with the verbatim v4
  code will show the header "Demo N / 7" (as in the shipped v4 files) and
  the v4 green error fill, while batch-A demos show "Demo NN / 42" and a
  red error fill. This was kept intentionally per the visual-identity
  instruction. If the parent wants "/ 42" everywhere, that is a separate
  one-line change to `build_demos_v4.py`, not a voice change.
- Per-chunk gates in the builder: duration over 0.5 s, mean volume above
  -40 dB, leading silence over 0.3 s trimmed keeping a 0.05 s pad, 4 s
  sleep between API calls, up to 3 retries with 45 s backoff on 429/5xx.

## Result per demo

### 01_XLOOKUP.mp4 — PASS (verified by coordinator)

- File: `output/demos_v5/01_XLOOKUP.mp4`, 708,503 bytes.
- Chunk durations (s): 12.24, 4.72, 9.40, 2.28, 5.52. Sum 34.16 s.
  Final file duration 34.37 s (difference 0.21 s, within the 0.5 s limit).
- Tokens (usageMetadata): prompt 126, audio 869. Estimated cost about $0.01.
- Trims applied: chunk 0 trimmed 0.32 s, chunk 4 trimmed 0.28 s of leading
  silence (0.05 s pad kept). No rewordings.
- Verification: ffprobe shows h264 1280x720 video plus AAC audio.
  silencedetect (noise=-40dB, d=2.0) found no internal gap over 2 s.
  volumedetect: mean -19.3 dB, max -1.5 dB. Frames viewed at focused
  (t=6 s), result (t=27.5 s), error (t=32 s): focused view shows Column A
  WorkID LOOKUP and Column F Owner RESULT with the B–E gap indicator and
  Ayesha Khan highlighted; result shows IT-001 -> Ayesha Khan with the
  correct formula bar; error shows IT-999 -> Not Found with the red banner.
  Headers not clipped, sheet tabs fully visible, cursor clear of key text.

### 02_VLOOKUP.mp4 — NOT BUILT (3 of 5 chunks cached, see below)

- Chunks 0–2 generated and verified healthy before the quota stop:
  10.80 s (prompt 42, audio 270), 4.64 s (19, 116), 10.32 s (34, 258).
  Chunks 3–4 missing. No video file was produced.

### 03_INDEX_MATCH.mp4 — NOT BUILT (2 of 5 chunks cached)

- Chunks 0–1 generated and verified healthy: 11.40 s (prompt 43, audio 285),
  9.84 s (32, 246). Chunks 2–4 missing. No video file was produced.

### 04_SUMIFS.mp4 — NOT BUILT (1 of 5 chunks cached)

- Chunk 0 generated and verified healthy: 14.44 s (prompt 49, audio 361),
  mean -20.5 dB, no silence gap over 2 s. Chunks 1–4 missing. No video
  file was produced.

### 05_MATCH.mp4, 06_SUMIF.mp4, 07_COUNTIF.mp4 — NOT BUILT

- 0 chunks. No files. The quota stopped the batch before they started.

### 08_COUNTIFS.mp4, 09_AVERAGEIF.mp4, 10_AVERAGEIFS.mp4, 11_IF.mp4 — NOT BUILT

- 0 chunks. No files. The demo 08 "IT-005" rewording is still pending
  (see above).

## Cached chunks for resume (verified healthy, reusable without new API calls)

In `~/workspace/excel-training/demo_work_v5/`, with matching
`chunk_B1A_*.raw.json` (token counts) and `chunk_B1A_*.txt` (chunk text):

- `B1A_02_0.mp3`, `B1A_02_1.mp3`, `B1A_02_2.mp3` (demo 02, chunks 0–2)
- `B1A_03_0.mp3`, `B1A_03_1.mp3` (demo 03, chunks 0–1)
- `B1A_04_0.mp3` (demo 04, chunk 0)
- Demo 01 chunks `B1A_01_0.mp3` … `B1A_01_4.mp3` plus `B1A_01_full.mp3`.

Note: `build_v5_batch1.py` regenerates every chunk from scratch. A resume
run should reuse these files (or the script must gain a cache check) or
the quota will be spent twice. Per-chunk chunk-text files let a resume
verify each cached chunk matches its text before reuse.

## Quota stop — exact evidence (why the batch stopped)

- A direct probe call after the stop returned HTTP 429 with:
  "Quota exceeded for metric:
  generativelanguage.googleapis.com/generate_content_free_tier_requests,
  limit: 10, model: gemini-3.8-flash-tts, description: Generate content
  requests per day limit exceeded on Free Tier. ... Please retry in
  1h20m18.551466849s."
- Observed: 13 successful generateContent calls happened on this key
  today (2 by the parent for the reference demos, 11 in this batch run),
  so the quota appears to refill on a rolling basis rather than as a hard
  daily reset; the probe's "retry in ~80 minutes" was measured at about
  11:44 PKT. Do NOT hammer retries: rejected calls do not help and the
  builder's 45 s retry loop already burned several attempts on demo 02.
- Diagnostic only (one call, NOT used in any deliverable, file deleted):
  `gemini-3.8-flash-lite-tts` with the same voice and style succeeded once
  (255 audio tokens), so the lite model has a separate free-tier quota.
  Switching models would change the locked voice spec; that decision
  belongs to the parent/user, not this batch. Alternatives for the parent:
  wait for quota refill, enable billing on the key's project, or approve
  the lite model.
- Token totals: deliverable (demo 01) prompt 126, audio 869. Cached
  chunks: demo 02 prompt 95 / audio 644; demo 03 prompt 75 / audio 531;
  demo 04 prompt 49 / audio 361. Diagnostic lite probe: prompt 42 /
  audio 255 (not a deliverable).

## Files written by this batch

- `output/demos_v5/01_XLOOKUP.mp4` (the only deliverable)
- `output/demos_v5/BUILD_REPORT_V5_BATCH1.md` (this file)
- `demo_work_v5/` (chunk WAV/MP3/JSON/TXT working files, `progress.json`)
- `build_v5_batch1.py` (builder script, workspace root of excel-training)


---

# Batch 2 — dual-model resume (demos 02–07 attempted, 2026-11-02)

Resume subagent, same day. Builder: `~/workspace/excel-training/build_v5_resume.py`
(cache-aware, idempotent, per-chunk `progress.json` recording). Nothing was
uploaded, moved, or deleted. `output/demos/`, `output/demos_v4/`, all backup
folders, and Drive were not touched.

## Voice / model policy change (authorized by Ali 2026-11-02 ~12:04 PKT)

- Ali allowed `gemini-3.8-flash-lite-tts` IN ADDITION TO
  `gemini-3.8-flash-tts`, always with the SAME voice (`Achird`) and the
  EXACT same style string. Rule: model chosen PER DEMO, never mixed inside
  one demo. If flash-tts quota is available, the whole demo uses flash-tts;
  only when flash-tts is quota-exhausted is a whole demo built with
  flash-lite-tts. No other model was used (no 2.5, no 3.1).
- Consequence: demos with cached flash-tts chunks (03, 04) MUST finish
  with flash-tts and are blocked while flash-tts is exhausted — they were
  NOT built with lite, to avoid mixing voices inside one demo. Demos with
  no cached chunks (05+) use lite while flash-tts is exhausted.
- The model used is recorded per demo in `demo_work_v5/progress.json`
  (`model` field, plus a per-chunk `models` list). A future run reuses the
  recorded model for a partial demo and never regenerates a cached chunk.
- Correct spellings in TTS input were kept (SUMIFS, COUNTIFS, SUMIF,
  COUNTIF, AVERAGEIF(S), TEXTJOIN, DATEDIF, EDATE, IT-005, IT-001 — never
  the Helio map spellings). The demo 08 chunk 3 "IT-005" restoration is
  coded in the resume builder (dry-checked) but demo 08 was not reached.

## Bug found and fixed during this run (no quota wasted)

- The first dual-model run blocked every fresh demo: `demo_record()` in
  `build_v5_resume.py` defaulted a new record's `model` to flash-tts, so
  the model-decision code read every untouched demo as "sticky flash" and
  raised DemoBlocked instead of falling back to lite. That run generated
  0 chunks (2 reused) and was discarded.
- Fix: new records default `model` to `None` (undecided); the 39 empty
  records the buggy run had created (demos 04–42, no chunk durations)
  were reset to `model: None`. Demos 01–03 kept their true flash-tts
  records. The builder was re-run after the fix. The fixed decision logic
  was dry-checked for demos 02–07 (cached/sticky -> flash; fresh ->
  flash, lite only when flash is exhausted).

## Result per demo (this run)

### 02_VLOOKUP.mp4 — PASS (built by the pre-update flash-only resume run, verified by the resume subagent)

- File: `output/demos_v5/02_VLOOKUP.mp4`, 748,599 bytes, 37.03 s.
- Chunk durations (s): 10.80, 4.64, 10.32, 3.23, 7.82. Sum 36.81 s
  (difference 0.22 s, within 0.5 s). Chunks 0–2 reused cached (Batch 1);
  chunks 3–4 generated new, all flash-tts / Achird.
- Tokens (chunk `*.raw.json` sums): prompt 134, audio 934.
  Cached portion (Batch 1): prompt 95, audio 644.
- Trims: none recorded for this demo (the pre-update run was stopped
  before it wrote its log; the chunk files passed the builder's
  duration/volume gates at generation).
- Verification (personally viewed): ffprobe h264 1280x720 + AAC;
  silencedetect (noise=-40dB, d=2.0) no gap over 2 s. Focused frame: data
  sheet, Column A WorkID / Column B Project, IT-001 -> Customer Portal
  Revamp. Result frame: `=VLOOKUP(B2,...)` with IT-001 ->
  Customer Portal Revamp. Error frame: IT-999 -> #N/A with the red
  banner and IFERROR fix. Header "Demo 02 / 42", tabs fully visible,
  nothing clipped. PASS.

### 03_INDEX_MATCH.mp4 — PARTIAL, 4 of 5 chunks (flash-tts, blocked on chunk 4)

- Chunks 0–1 reused cached (11.40 s, 9.84 s). Chunk 2 generated
  5.72 s (leading-silence trim 0.28 s, 0.05 s pad kept). Chunk 3 needed a
  45 s-backoff retry (attempt 0 quota, retry 1 succeeded — rolling refill
  freed one call), generated 3.52 s. Chunk 4: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED -> demo left partial, no video file.
- Chunk durations so far: 11.40, 9.84, 5.72, 3.52. Tokens so far:
  prompt 112, audio 769. Model: flash-tts (sticky, cached chunks).
- Resume: generate chunk 4 ONLY, with flash-tts, after quota refill.

### 04_SUMIFS.mp4 — NOT BUILT, 1 of 5 chunks cached (flash-tts, blocked)

- Chunk 0 cached from Batch 1: 14.44 s, prompt 49, audio 361, mean
  -20.5 dB. This run re-checked its gates and recorded it into
  `progress.json` (provenance `cached-batch1`, model flash-tts) so the
  resume record is complete. Chunks 1–4 missing; the demo raised
  DemoBlocked (requires flash-tts, flash-tts exhausted) with no API call.
  No video file.
- Resume: generate chunks 1–4 ONLY, with flash-tts, after quota refill.

### 05_MATCH.mp4 — PASS (complete, flash-lite-tts)

- File: `output/demos_v5/05_MATCH.mp4`, 1,048,205 bytes, 55.23 s.
- Chunk durations (s): 10.84, 8.24, 18.44, 9.12, 8.36. Sum 55.00 s
  (difference 0.23 s). All 5 chunks generated new, lite model, no reuse.
  No trims. Chunk 4 needed a 45 s-backoff retry (attempt 0 quota, retry 1
  succeeded — rolling refill).
- Tokens: prompt 181, audio 1376.
- Verification (personally viewed): ffprobe h264 1280x720 + AAC;
  silencedetect no gap over 2 s. Focused frame: Position / WorkID /
  Project columns, rows 1–5, IT-005 highlighted. Result frame:
  `=MATCH(B2,...)`, IT-005 -> position 5. Error frame: IT-999 -> #N/A
  with the red banner and IFERROR fix. Header "Demo 5 / 7" and green
  error-cell fill are the verbatim v4 rendering for demos 05–07 (known
  visual-identity note from Batch 1, kept intentionally). Tabs fully
  visible, nothing clipped. PASS.

### 06_SUMIF.mp4 — PASS (complete, flash-lite-tts)

- File: `output/demos_v5/06_SUMIF.mp4`, 859,094 bytes, 44.92 s.
- Chunk durations (s): 16.68, 5.33, 14.36, 4.44, 3.88. Sum 44.69 s
  (difference 0.23 s). All 5 chunks generated new, lite model.
  Trim: chunk 1, 0.27 s leading silence (0.05 s pad kept).
- Tokens: prompt 143, audio 1125.
- Verification (personally viewed): ffprobe h264 1280x720 + AAC;
  silencedetect no gap over 2 s. Focused frame: Column B Project /
  Column J Actual Hours, HRMS Upgrade highlighted. Result frame:
  `=SUMIF(...)`, HRMS Upgrade -> 312.3 hours. Error frame:
  Wrong Project -> 0 with the red banner. Header "Demo 6 / 7" (verbatim
  v4 rendering, see note above). Tabs fully visible, nothing clipped.
  PASS.

### 07_COUNTIF.mp4 — PARTIAL, 3 of 5 chunks (flash-lite-tts, blocked on chunk 3)

- Chunks generated new, lite model: 10.68 s, 4.84 s, 9.54 s
  (trim 0.30 s on chunk 2). Tokens so far: prompt 86, audio 634.
  Chunk 3: 3 attempts, all HTTP 429 RESOURCE_EXHAUSTED -> both models
  exhausted -> run stopped cleanly. No video file.
- Resume: generate chunks 3–4 ONLY, with flash-lite-tts (sticky).

### 08–42 — NOT ATTEMPTED in this run

- 0 chunks, no files. The stop was reached at demo 07. Next run order:
  03 chunk 4 (flash), 04 chunks 1–4 (flash), 07 chunks 3–4 (lite),
  then 08–42 in order (flash first per demo, lite only when flash is
  exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 03 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day limit
  exceeded on Free Tier."
- Lite-tts: demo 07 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed twice in this run: a single call freed up
  after a 45 s backoff (demo 03 chunk 3 retry 1, demo 05 chunk 4
  retry 1). No retries were burned in a loop; the run stopped as soon
  as both models reported exhausted.
- Calls this run (from `resume_results.json`): 15 generated
  (2 flash-tts + 13 flash-lite-tts), 2 reused, 0 non-quota failures.
  New audio tokens this run: 3373 (03: 238, 05: 1376, 06: 1125,
  07: 634).

## Token totals (all v5 work so far, from `demo_work_v5/chunk_*.raw.json`)

- Audio tokens 6068, prompt tokens 831, across 28 chunk files:
  01: 869/126 (5 chunks), 02: 934/134 (5), 03: 769/112 (4),
  04: 361/49 (1), 05: 1376/181 (5), 06: 1125/143 (5), 07: 634/86 (3).
  (Per-demo `progress.json` `tokens` fields hold only the last recorded
  chunk's counts; the `*.raw.json` files are the per-chunk source of
  truth.)

## Resume state (for the next run / cron)

- `demo_work_v5/progress.json` records every finished chunk (demo,
  chunk index, text hash, file, duration, trim, provenance, model) and
  is saved after every chunk. Complete demos 01, 02, 05, 06 are skipped
  on re-run. Partial: 03 (4/5, flash), 04 (1/5, flash), 07 (3/5, lite).
  Untouched: 08–42.
- Re-running `build_v5_resume.py` regenerates NOTHING already present:
  it reuses verified cached chunk MP3s, generates only missing chunks
  with each demo's recorded model, and stops cleanly if both models
  are exhausted.
- Run artifacts: `demo_work_v5/resume_log.txt`,
  `demo_work_v5/resume_results.json`, frame checks in
  `demo_work_v5/verify_{02,05,06}/`.

## Files written in Batch 2

- `output/demos_v5/05_MATCH.mp4`, `output/demos_v5/06_SUMIF.mp4`
  (02_VLOOKUP.mp4 was written by the pre-update run and verified here)
- `output/demos_v5/BUILD_REPORT_V5_BATCH1.md` (this file, Batch 2
  section appended; Batch 1 evidence above untouched)
- `build_v5_resume.py` (dual-model fix), `demo_work_v5/progress.json`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`,
  chunk working files in `demo_work_v5/`


---

# Batch 3 — cron resume run 2026-11-02 13:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes. Nothing was
uploaded, moved, or deleted except the single Drive upload of
07_COUNTIF.mp4 recorded below. `output/demos/`,
`output/demos_v4/`, and all backup folders were not touched.

## Result per demo (this run)

### 03_INDEX_MATCH.mp4 — STILL PARTIAL, 4 of 5 chunks (flash-tts)

- Chunks 0–3 reused cached (11.40, 9.84, 5.72, 3.52 s). Chunk 4:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (flash-tts free-tier
  requests per day exceeded). No video file. Resume: chunk 4 ONLY,
  flash-tts, after quota refill. Sticky flash-tts; do NOT build with lite.

### 04_SUMIFS.mp4 — STILL NOT BUILT, 1 of 5 chunks (flash-tts)

- Chunk 0 cached (14.44 s). DemoBlocked with no API call: requires
  flash-tts, flash-tts exhausted. Resume: chunks 1–4 ONLY, flash-tts.

### 07_COUNTIF.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/07_COUNTIF.mp4`, 706,694 bytes, 35.51 s.
- Chunk durations (s): 10.68, 4.84, 9.54, 5.26, 4.96. Sum 35.28 s
  (difference 0.23 s, within 0.5 s). Chunks 0–2 reused cached;
  chunks 3–4 generated new this run, lite model, Achird, same style.
  New-chunk tokens (raw JSON): prompt 32, audio 263
  (chunk 3: 15/139; chunk 4: 17/124). Trim: chunk 3, 0.30 s leading
  silence (0.05 s pad kept, recorded by the builder).
- Verification (personally viewed this run): ffprobe h264 1280x720 +
  AAC; silencedetect (noise=-40dB, d=2.0) no gap over 2 s;
  volumedetect mean -20.0 dB, max -1.5 dB. Focused frame: Column A
  WorkID / Column G Status - LOOKUP, Delayed highlighted. Result
  frame: `=COUNTIF('1_IT_PM_DATA'!G4:G123,B2)`, Delayed -> 18.
  Error frame: Wrong Status -> 0 with the red banner. Header
  "Demo 7 / 7" and green error-cell fill are the verbatim v4
  rendering for demos 05–07 (known visual-identity note, kept).
  Tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 08_COUNTIFS.mp4 — PARTIAL, 2 of 5 chunks (flash-lite-tts)

- Chunks generated new this run, lite model: chunk 0, 6.52 s
  (prompt 23, audio 163); chunk 1, 10.12 s (prompt 35, audio 253).
  Correct spelling COUNTIFS kept. Chunk 2: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (lite-tts also exhausted) -> run stopped cleanly.
  No video file. The pending chunk-2 text file is present in
  `demo_work_v5/`; chunk 3 "IT-005" restoration is coded in the
  resume builder and was not reached. Resume: chunks 2–4 ONLY,
  flash-lite-tts (sticky, recorded in progress.json).

### 09–42 — NOT ATTEMPTED in this run

- 0 chunks, no files. Next run order: 03 chunk 4 (flash), 04 chunks
  1–4 (flash), 08 chunks 2–4 (lite), then 09–42 in order (flash
  first per demo, lite only when flash is exhausted and the demo has
  no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 03 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 08 chunk 2, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Calls this run (from the builder log): 4 generated (all lite),
  7 reused, 0 non-quota failures. New audio tokens this run: 679
  (07: 263, 08: 416).

## Drive sync (this run)

- Folder listing checked first (13:41 PKT): only 01, 02, 05, 06
  present as video/mp4. 07_COUNTIF.mp4 was NOT present, so only it
  was uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 07_COUNTIF.mp4`
- Post-upload listing re-checked: 01, 02, 05, 06, 07 present with
  matching sizes (07: 706,694 bytes). No file was moved, deleted,
  or trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 05 (lite),
  06 (lite), 07 (lite) — 5 of 42 demos.
- Partial: 03 (4/5, flash), 04 (1/5, flash), 08 (2/5, lite).
  Untouched: 09–42. `demo_work_v5/progress.json` records every
  finished chunk and is the source of truth; the resume builder
  regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_1335.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_07/`.


---

# Batch 4 — cron resume run 2026-11-02 15:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
13:41 PKT). Nothing was uploaded, moved, or deleted except the Drive
uploads recorded below. `output/demos/`, `output/demos_v4/`, and all
backup folders were not touched. Progress backup:
`demo_work_v5/progress.json.bak-1505` (taken before the run).

## Result per demo (this run)

All 7 newly finished demos were independently re-verified by the
coordinator after the builder run (ffprobe + silencedetect +
volumedetect + frames viewed as montages in
`demo_work_v5/verify_montage_{03,04,08,09,10,12,13}.png`). Token
totals below are sums of the per-chunk `*.raw.json` usageMetadata
(the source of truth); `progress.json` `tokens` fields double-count
on cache reuse and are NOT used here.

### 03_INDEX_MATCH.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/03_INDEX_MATCH.mp4`, 768,069 bytes, 38.38 s.
- Chunk durations (s): 11.40, 9.84, 5.72, 3.52, 7.68. Sum 38.16 s
  (difference 0.22 s, within 0.5 s). Chunks 0–3 reused cached;
  chunk 4 generated new this run, flash-tts / Achird, same style.
- Tokens (all 5 chunks, raw JSON): prompt 142, audio 961.
- Verification: ffprobe h264 1280x720 + AAC; silencedetect
  (noise=-40dB, d=2.0) no gap over 2 s; volumedetect mean -20.1 dB,
  max -1.3 dB. Focused frame: Column A WorkID / Column M DueDate,
  IT-001 -> 20 May 2025. Result frame: INDEX-MATCH formula,
  IT-001 -> 20 May 2025. Error frame: IT-999 -> #N/A with red
  banner. Header "Demo 03 / 42", tabs fully visible, nothing clipped.

### 04_SUMIFS.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/04_SUMIFS.mp4`, 719,306 bytes, 33.72 s.
- Chunk durations (s): 14.44, 7.92, 3.70, 2.88, 4.56. Sum 33.50 s
  (difference 0.22 s). Chunk 0 reused cached; chunks 1–4 generated
  new, flash-tts. Trims: chunk 2, 0.30 s; chunk 3, 0.28 s leading
  silence (0.05 s pad kept).
- Tokens (all 5 chunks, raw JSON): prompt 112, audio 852.
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -19.8 dB, max -1.7 dB. Focused: Project / Sprint /
  Actual Hours columns. Result: `=SUMIFS(...)`, HRMS Upgrade,
  Sprint 2 -> 96.5 hours. Error: Wrong Project -> 0 with red
  banner. Header "Demo 04 / 42", tabs visible, nothing clipped.

### 08_COUNTIFS.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/08_COUNTIFS.mp4`, 914,896 bytes, 47.35 s.
- Chunk durations (s): 6.52, 10.12, 11.08, 8.44, 10.97. Sum 47.13 s
  (difference 0.22 s). Chunks 0–1 reused cached; chunks 2–4
  generated new, lite model (sticky, recorded in progress.json).
  Chunk 3 text keeps the restored "IT-005". Trim: chunk 4, 0.27 s.
- Tokens (all 5 chunks, raw JSON): prompt 157, audio 1185.
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -19.2 dB, max -1.7 dB. Focused: Status / Priority
  columns, Delayed + Critical highlighted. Result: COUNTIFS ->
  count 5. Error: mismatched range sizes -> #VALUE! with red
  banner. Header "Demo 08 / 42", tabs visible, nothing clipped.

### 09_AVERAGEIF.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/09_AVERAGEIF.mp4`, 787,391 bytes, 40.48 s.
- Chunk durations (s): 6.44, 7.12, 5.00, 6.74, 14.94. Sum 40.24 s
  (difference 0.24 s). All 5 chunks generated new, flash-tts.
  Trims: chunk 3, 0.26 s; chunk 4, 0.35 s.
- Tokens (raw JSON): prompt 139, audio 1021.
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -19.4 dB, max -1.4 dB. Focused: Status / Actual Hours.
  Result: AVERAGEIF, Completed -> 13.025 hours. Error: Wrong Status
  -> #DIV/0! with red banner. Header "Demo 09 / 42", tabs visible.

### 10_AVERAGEIFS.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/10_AVERAGEIFS.mp4`, 940,648 bytes, 45.67 s.
- Chunk durations (s): 7.08, 10.52, 7.35, 6.68, 13.80. Sum 45.43 s
  (difference 0.24 s). All 5 chunks generated new, flash-tts.
  Chunk 1 and chunk 3 each needed a 45 s-backoff retry (rolling
  refill freed a call). Trim: chunk 2, 0.25 s.
- Tokens (raw JSON): prompt 148, audio 1142.
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -18.8 dB, max -1.1 dB. Focused: Project / Status /
  Variance Hours. Result: AVERAGEIFS, Customer Portal Revamp +
  Delayed -> 3.7 hours. Error: Wrong Project -> #DIV/0! with red
  banner. Header "Demo 10 / 42", tabs visible.

### 12_IFS.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/12_IFS.mp4`, 990,857 bytes, 51.31 s.
- Chunk durations (s): 8.28, 11.32, 7.86, 7.92, 15.72. Sum 51.10 s
  (difference 0.21 s). All 5 chunks generated new, lite model
  (flash-tts was exhausted when demo 11 stopped; fresh demo, so
  lite per the per-demo rule). Trim: chunk 2, 0.30 s.
- Tokens (raw JSON): prompt 158, audio 1285.
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -18.7 dB, max -1.3 dB. Focused: Status / Variance /
  RAG Rating. Result: IFS, Delayed + 5.0 -> Red. Error: Blocked +
  -2.5 with no TRUE default -> #N/A with red banner. Header
  "Demo 12 / 42", tabs visible.

### 13_AND_OR.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/13_AND_OR.mp4`, 940,883 bytes, 48.48 s.
- Chunk durations (s): 9.40, 9.32, 9.28, 7.52, 12.72. Sum 48.24 s
  (difference 0.24 s). All 5 chunks generated new, lite model.
  Several chunks needed 45 s-backoff retries (rolling refill).
  No trims.
- Tokens (raw JSON): prompt 158, audio 1206.
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -20.1 dB, max -1.2 dB. Focused: Status / Priority /
  Action, Delayed + Critical -> ESCALATE. Result: AND formula,
  ESCALATE. Error frame (v4 visual, kept verbatim): "Delayd"
  spelling -> Monitor with red banner. Header "Demo 13 / 42",
  tabs visible.

### 11_IF.mp4 — PARTIAL, 2 of 5 chunks (flash-tts, blocked)

- Chunks 0–1 generated this run, flash-tts: 6.32 s (prompt 21,
  audio 158), 12.32 s (prompt 40, audio 308). Chunk 2: 3 attempts,
  all HTTP 429 RESOURCE_EXHAUSTED -> demo left partial, no video
  file. Sticky flash-tts; do NOT build with lite. Resume: chunks
  2–4 ONLY, flash-tts, after quota refill.

### 14_IFERROR.mp4 — PARTIAL, 1 of 5 chunks (flash-lite-tts, blocked)

- Chunk 0 generated this run, lite model: 7.08 s (prompt 26,
  audio 177). Chunk 1: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (lite also exhausted) -> run stopped cleanly.
  No video file. Resume: chunks 1–4 ONLY, flash-lite-tts (sticky).

### 15–42 — NOT ATTEMPTED in this run

- 0 chunks, no files. Next run order: 11 chunks 2–4 (flash),
  14 chunks 1–4 (lite), then 15–42 in order (flash first per
  demo, lite only when flash is exhausted and the demo has no
  flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 11 chunk 2, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 14 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed repeatedly in this run: single calls
  freed up after 45 s backoffs (demos 10 and 13). No retries were
  burned in a loop beyond the builder's 3-attempt rule; the run
  stopped as soon as both models reported exhausted.
- Calls this run (from `resume_results.json`, overwritten by this
  run): 31 generated (17 flash-tts + 14 flash-lite-tts), 7 reused,
  0 non-quota failures. New audio tokens this run (generated
  chunks, from the builder log / raw JSON): 03: 192, 04: 491,
  08: 769, 09: 1021, 10: 1142, 11: 466, 12: 1285, 13: 1206,
  14: 177 — total 6,749 audio tokens for the 31 generated chunks
  (the 7 reused chunks added no new calls).

## Drive sync (this run)

- Folder listing checked first (15:08 PKT, before the build
  finished, and again at 15:27 PKT before uploading): only 01,
  02, 05, 06, 07 present as video/mp4, with matching local
  sizes. The 7 newly finished demos were NOT present, so only
  they were uploaded after verification, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name <file>`.
  Uploads returned Drive file IDs: 03 `15Oa6HWRvAAGfDaA3vKOg8mbXWbuI3vMp`,
  04 `1ca-oef2fOShX2yMskuTsEkqguM8GiARs`,
  08 `1PMLACW8bfl25pQFrds2S4JNeRGZ_oE66`,
  09 `1jzt_K4nliTlEEV1_uqbFvWGRVyHDgcXN`,
  10 `1Sr7g1k0bsxIGBAWm98l_P4HRleZjcccv`,
  12 `1XKe4Veqiy3Cli2pm06A4Pdnw6xY1QlCU`,
  13 `1GprlVQQOwV7OB6HjXIqoD5vr6r04a_Zq`.
- Post-upload listing re-checked at 15:27 PKT: 01–04, 05–10,
  12, 13 present (12 mp4s) with sizes matching the local files
  exactly. No file was moved, deleted, or trashed in Drive;
  backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 12 (lite), 13 (lite) — 12 of
  42 demos.
- Partial: 11 (2/5, flash), 14 (1/5, lite). Untouched: 15–42.
  `demo_work_v5/progress.json` records every finished chunk and
  is the source of truth; the resume builder regenerates nothing
  already present.
- Run artifacts: `demo_work_v5/cron_run_1505.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_{03,04,08,09,10,12,13}/` and montages
  `demo_work_v5/verify_montage_{03,04,08,09,10,12,13}.png`.


---

# Batch 5 — cron resume run 2026-11-02 16:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
15:27 PKT). Nothing was uploaded, moved, or deleted except the
single Drive upload of 14_IFERROR.mp4 recorded below.
`output/demos/`, `output/demos_v4/`, and all backup folders were
not touched. Progress backup:
`demo_work_v5/progress.json.bak-1635` (taken before the run).

## Result per demo (this run)

### 14_IFERROR.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/14_IFERROR.mp4`, 705,867 bytes, 35.03 s.
- Chunk durations (s): 7.08, 10.00, 4.64, 8.72, 4.36. Sum 34.80 s
  (difference 0.23 s, within 0.5 s). Chunk 0 reused cached;
  chunks 1–4 generated new this run, lite model, Achird, same
  style. Chunk 1 and chunk 4 each needed a 45 s-backoff retry
  (rolling refill freed a call). No trims recorded.
- Tokens (all 5 chunks, raw JSON): prompt 115, audio 872
  (new chunks this run: prompt 89, audio 695).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -20.0 dB, max -1.5 dB. Focused frame:
  Column A WorkID / Column B Project, IT-001 -> Customer Portal
  Revamp. Result frame: `=IFERROR(VLOOKUP(B2,...),"Not Found")`,
  IT-999 -> Not Found. Error frame: raw VLOOKUP -> #N/A with
  the red banner. Header "Demo 14 / 42", tabs fully visible,
  nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 11_IF.mp4 — STILL PARTIAL, 3 of 5 chunks (flash-tts)

- Chunks 0–1 reused cached (6.32 s, 12.32 s). Chunk 2 generated
  this run, flash-tts: 5.96 s (prompt 20, audio 149), after two
  45 s-backoff retries on quota. Chunk 3: 3 attempts, all HTTP
  429 RESOURCE_EXHAUSTED -> demo left partial, no video file.
  Text: "Formula mein, agar Variance teen se zyada hai, to
  Overdue, warna On Track." Sticky flash-tts; do NOT build with
  lite. Resume: chunks 3–4 ONLY, flash-tts, after quota refill.

### 15_TEXT.mp4 — NOT ATTEMPTED (0 chunks; lite quota exhausted on chunk 0)

- Demo 15 was reached with flash-tts already exhausted, so the
  builder recorded it as lite before its first call; chunk 0:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (lite also
  exhausted) -> run stopped cleanly. No chunk was generated.
  Its `progress.json` model was reset to None (undecided) after
  the run, with no chunk data lost, so the next run re-decides
  flash-first per the per-demo model order instead of treating
  the failed first call as a sticky lite choice.

### 16–42 — NOT ATTEMPTED in this run

- 0 chunks, no files. Next run order: 11 chunks 3–4 (flash),
  then 15–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 11 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per
  day limit exceeded on Free Tier."
- Lite-tts: demo 15 chunk 0, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 11 chunk 2 retry 2, demo 14 chunks 1 and
  4 retry 1). No retries were burned beyond the builder's
  3-attempt rule; the run stopped as soon as both models
  reported exhausted.
- Calls this run (from `resume_results.json`): 5 generated
  (1 flash-tts + 4 flash-lite-tts), 3 reused, 0 non-quota
  failures. New audio tokens this run: 844 (prompt 109).

## Drive sync (this run)

- Folder listing checked first (16:45 PKT): 12 mp4s present
  (01–10, 12, 13), matching local sizes. 14_IFERROR.mp4 was
  NOT present, so only it was uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 14_IFERROR.mp4`
  Upload returned Drive file ID
  `1dEJE0Vr_fysly3ff-ZQCI4GDry_XAyRC`.
- Post-upload listing re-checked: 13 mp4s (01–10, 12–14) with
  sizes matching the local files exactly (14: 705,867 bytes).
  No file was moved, deleted, or trashed in Drive; backup
  folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 12 (lite), 13 (lite), 14
  (lite) — 13 of 42 demos.
- Partial: 11 (3/5, flash). Untouched: 15–42 (15 model reset
  to undecided, see above). `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_1635.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_14/`.


---

# Batch 6 — cron resume run 2026-11-02 18:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
16:47 PKT). Nothing was uploaded, moved, or deleted except the
single Drive upload of 11_IF.mp4 recorded below.
`output/demos/`, `output/demos_v4/`, and all backup folders were
not touched. Progress backup:
`demo_work_v5/progress.json.bak-1805` (taken before the run).

## Runtime restart during this run (no work lost, no chunk regenerated)

- The first builder invocation (18:05 PKT, log
  `demo_work_v5/cron_run_1805.log`) was killed by a runtime restart
  at about 18:08 PKT: its exec log is empty (0 bytes, tee output
  lost with the kill), `resume_results.json` / `resume_log.txt`
  were never written, and no demo was finished by it. Its only
  effects were progress.json writes recording demo 15 as
  sticky lite with 0 chunks (model set before its first call,
  which never produced audio) and the demo 11 chunk 3 / demo 15
  chunk 0 text files. No chunk audio was generated twice because
  of the kill: the second invocation reused the cache checks.
- Before the second invocation, demo 15's model was reset to
  None (undecided, models []) — the same Batch 5 treatment for a
  0-chunk sticky record — so it re-decided flash-first. The
  second invocation (`python3 -u build_v5_resume.py`, log
  `demo_work_v5/cron_run_1805b.log`) survived, finished cleanly
  on quota exhaustion at 18:17 PKT (BUILDER_EXIT:0), and wrote
  `resume_log.txt` / `resume_results.json`. All results below are
  from the second invocation; the first invocation generated no
  deliverable audio that was counted.

## Result per demo (this run)

### 11_IF.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/11_IF.mp4`, 817,805 bytes, 42.88 s.
- Chunk durations (s): 6.32, 12.32, 5.96, 5.76, 12.28. Sum 42.64 s
  (difference 0.24 s, within 0.5 s). Chunks 0–2 reused cached;
  chunks 3–4 generated new this run, flash-tts / Achird, same
  style. No trims.
- Tokens (all 5 chunks, raw JSON — the source of truth;
  `progress.json` `tokens` for this demo double-counts on cache
  reuse and is NOT used here): prompt 147, audio 1066 (new chunks
  this run: chunk 3 prompt 19 / audio 144; chunk 4 prompt 47 /
  audio 307).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.8 dB, max -0.7 dB. Focused frame:
  Column A WorkID / Column K Variance Hours - LOOKUP, IT-005
  highlighted, +5.0. Result frame: `=IF(B2>3,"Overdue","On
  Track")`, 5.0 -> Overdue. Error frame: reversed sign
  `=IF(B2<3,...)` -> On Track with the red banner. Header
  "Demo 11 / 42", tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 15_TEXT.mp4 — PARTIAL, 1 of 5 chunks (flash-tts, blocked)

- Chunk 0 generated this run, flash-tts: 7.40 s (prompt 21,
  audio 185). Text: "Is demo mein hum TEXT seekhenge. TEXT date
  ya number ko formatted text mein badalta hai." Chunk 1:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (flash-tts) ->
  demo left partial, no video file. Sticky flash-tts; do NOT
  build with lite. Resume: chunks 1–4 ONLY, flash-tts, after
  quota refill.

### 16_CONCAT.mp4 — PARTIAL, 2 of 5 chunks (flash-lite-tts, blocked)

- Chunks 0–1 generated this run, lite model (flash-tts was
  exhausted when demo 15 stopped; fresh demo, so lite per the
  per-demo rule): 7.08 s (prompt 23, audio 177), 5.92 s
  (prompt 14, audio 148). Correct spelling IT-001 restored in
  chunk 1 (Helio artifact removed, recorded in progress.json).
  Chunk 2: 3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (lite
  also exhausted) -> run stopped cleanly. No video file.
  Resume: chunks 2–4 ONLY, flash-lite-tts (sticky).

### 17–42 — NOT ATTEMPTED in this run

- 0 chunks, no files. Next run order: 15 chunks 1–4 (flash),
  16 chunks 2–4 (lite), then 17–42 in order (flash first per
  demo, lite only when flash is exhausted and the demo has no
  flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 15 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 16 chunk 2, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- No rolling-refill success was observed in this run's retries;
  the run stopped as soon as both models reported exhausted.
- Calls this run (from `resume_results.json`, second
  invocation): 5 generated (3 flash-tts + 2 flash-lite-tts),
  3 reused, 0 non-quota failures. New audio tokens this run:
  961 (11: 451, 15: 185, 16: 325; prompt 124).

## Drive sync (this run)

- Folder listing checked first (18:17 PKT): 13 mp4s present
  (01–10, 12–14), matching local sizes. 11_IF.mp4 was NOT
  present, so only it was uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 11_IF.mp4`
  Upload returned Drive file ID
  `17Sp6yO18ERkRTcMksPJbABIYJTkjTvwj`.
- Post-upload listing re-checked: 14 mp4s (01–14) with sizes
  matching the local files exactly (11: 817,805 bytes). No file
  was moved, deleted, or trashed in Drive; backup folders
  untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite) — 14 of 42 demos.
- Partial: 15 (1/5, flash), 16 (2/5, lite). Untouched: 17–42.
  `demo_work_v5/progress.json` records every finished chunk and
  is the source of truth; the resume builder regenerates nothing
  already present.
- Run artifacts: `demo_work_v5/cron_run_1805.log` (empty —
  killed with the runtime restart, see above),
  `demo_work_v5/cron_run_1805b.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_11/`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 7 — cron resume run 2026-11-02 19:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
18:18 PKT). Nothing was uploaded, moved, or deleted except the
single Drive upload of 16_CONCAT.mp4 recorded below.
`output/demos/`, `output/demos_v4/`, and all backup folders were
not touched. Progress backup:
`demo_work_v5/progress.json.bak-1935` (taken before the run).

## Result per demo (this run)

### 16_CONCAT.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/16_CONCAT.mp4`, 687,624 bytes, 33.86 s.
- Chunk durations (s): 7.08, 5.92, 6.76, 7.40, 6.48. Sum 33.64 s
  (difference 0.22 s, within 0.5 s). Chunks 0–1 reused cached;
  chunks 2–4 generated new this run, lite model, Achird, same
  style. No trims. Chunk 2 needed two 45 s-backoff retries and
  chunk 4 two 45 s-backoff retries (rolling refill freed a call
  each time). Chunk texts verbatim as recorded in progress.json;
  chunk 2 keeps the restored "IT-001" (Helio artifact removed).
- Tokens (all 5 chunks, raw JSON): prompt 106, audio 841
  (new chunks this run: prompt 69, audio 516).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -20.8 dB, max -1.3 dB. Focused frame:
  Column A WorkID / Column B Project - LOOKUP, IT-001 ->
  Customer Portal Revamp highlighted. Result frame:
  `='1_IT_PM_DATA'!A4&" - "&'1_IT_PM_DATA'!B4`, IT-001 -
  Customer Portal Revamp. Error frame: date joined directly ->
  IT-001 - 45797 with the red banner and TEXT fix. Header
  "Demo 16 / 42", tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 15_TEXT.mp4 — PARTIAL, 2 of 5 chunks (flash-tts)

- Chunk 0 reused cached (7.40 s). Chunk 1 generated this run,
  flash-tts: 8.36 s (prompt 15, audio 209), after one 45 s-backoff
  retry on quota. Text: "Formula mein Due Date, aur format, MMM
  dash YYYY." Chunk 2: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED -> demo left partial, no video file.
  Sticky flash-tts; do NOT build with lite. Resume: chunks 2–4
  ONLY, flash-tts, after quota refill.

### 17_LEFT_MID_RIGHT — NOT ATTEMPTED (0 chunks; both quotas exhausted)

- Demo 17 was reached with flash-tts already exhausted, so the
  builder recorded it as lite before its first call; chunk 0:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (lite also
  exhausted) -> run stopped cleanly. No chunk was generated.
  Its `progress.json` model was reset to None (undecided) after
  the run, with no chunk data lost, so the next run re-decides
  flash-first per the per-demo model order instead of treating
  the failed first call as a sticky lite choice. (Same treatment
  as demo 15 in Batch 5 / Batch 6.)

### 18–42 — NOT ATTEMPTED in this run

- 0 chunks, no files. Next run order: 15 chunks 2–4 (flash),
  then 17–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 15 chunk 2, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 17 chunk 0, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 15 chunk 1 retry 1, demo 16 chunks 2 and
  4 retry 2). No retries were burned beyond the builder's
  3-attempt rule; the run stopped as soon as both models
  reported exhausted.
- Calls this run (from `resume_results.json`): 4 generated
  (1 flash-tts + 3 flash-lite-tts), 3 reused, 0 non-quota
  failures. New audio tokens this run: 725 (15: 209, 16: 516;
  prompt 84).

## Drive sync (this run)

- Folder listing checked first (19:36 PKT, before the build,
  and again at 19:44 PKT before uploading): 14 mp4s present
  (01–14), matching local sizes. 16_CONCAT.mp4 was NOT present,
  so only it was uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 16_CONCAT.mp4`
  Upload returned Drive file ID
  `1D0mxs_Wkm3p1ApBbPdvPKrjvIosmDAsO`.
- Post-upload listing re-checked: 15 mp4s (01–14, 16) with
  sizes matching the local files exactly (16: 687,624 bytes).
  Demo 15 is correctly absent (still partial). No file was
  moved, deleted, or trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 16 (lite) — 15 of 42 demos.
- Partial: 15 (2/5, flash). Untouched: 17–42 (17 model reset
  to undecided, see above). `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_1935.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_16/`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 8 — cron resume run 2026-11-02 21:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
19:42 PKT). Nothing was uploaded, moved, or deleted; no new video
file was produced. `output/demos/`, `output/demos_v4/`, and all
backup folders were not touched. Progress backup:
`demo_work_v5/progress.json.bak-2105` (taken before the run).

## Result per demo (this run)

### 15_TEXT.mp4 — STILL PARTIAL, 3 of 5 chunks (flash-tts)

- Chunks 0–1 reused cached (7.40 s, 8.36 s). Chunk 2 generated
  this run, flash-tts: 5.64 s (prompt 23, audio 141), after one
  45 s-backoff retry on quota. Text: "Result, May, do hazar
  pachees. Ye report title ke liye behtareen hai." Chunk 3:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED -> demo left
  partial, no video file. Sticky flash-tts; do NOT build with
  lite. Resume: chunks 3–4 ONLY, flash-tts, after quota refill.

### 17_LEFT_MID_RIGHT — PARTIAL, 1 of 5 chunks (flash-lite-tts)

- Fresh demo, reached with flash-tts already exhausted, so the
  builder chose lite for the whole demo per the per-demo rule.
  Chunk 0 generated this run, lite model: 7.36 s (prompt 23,
  audio 184), after two 45 s-backoff retries on quota. Text:
  "Is demo mein hum LEFT, MID aur RIGHT seekhenge. Ye text ka
  hissa nikalte hain." Chunk 1: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (lite also exhausted) -> run stopped
  cleanly. No video file. The 1-chunk lite record is sticky
  and is NOT reset (unlike a 0-chunk record): resume chunks
  1–4 ONLY, flash-lite-tts.

### 18–42 — NOT ATTEMPTED in this run

- 0 chunks, no files. Next run order: 15 chunks 3–4 (flash),
  17 chunks 1–4 (lite), then 18–42 in order (flash first per
  demo, lite only when flash is exhausted and the demo has no
  flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 15 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 17 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 15 chunk 2 retry 1, demo 17 chunk 0
  retry 2). No retries were burned beyond the builder's
  3-attempt rule; the run stopped as soon as both models
  reported exhausted.
- Calls this run (from `resume_results.json`): 2 generated
  (1 flash-tts + 1 flash-lite-tts), 2 reused, 0 non-quota
  failures. New audio tokens this run: 325 (15: 141, 17: 184;
  prompt 46). Builder exit 0.

## Drive sync (this run)

- None. No demo finished this run, so no folder listing check
  or upload was needed and none was made. Drive still holds
  the same 15 mp4s (01–14, 16) from earlier runs.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 16 (lite) — 15 of 42 demos.
- Partial: 15 (3/5, flash), 17 (1/5, lite). Untouched: 18–42.
  `demo_work_v5/progress.json` records every finished chunk and
  is the source of truth; the resume builder regenerates nothing
  already present.
- Run artifacts: `demo_work_v5/cron_run_2105.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run).
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 9 — cron resume run 2026-11-02 22:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
21:12 PKT in UTC listing terms, 16:12 UTC). Nothing was uploaded,
moved, or deleted except the two Drive uploads recorded below.
`output/demos/`, `output/demos_v4/`, and all backup folders were
not touched. Progress backup:
`demo_work_v5/progress.json.bak-2235` (taken before the run).

## Result per demo (this run)

### 15_TEXT.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/15_TEXT.mp4`, 671,711 bytes, 33.90 s.
- Chunk durations (s): 7.40, 8.36, 5.64, 8.40, 3.88. Sum 33.68 s
  (difference 0.22 s, within 0.5 s). Chunks 0–2 reused cached;
  chunks 3–4 generated new this run, flash-tts / Achird, same
  style. Chunk 4 needed two 45 s-backoff retries (rolling refill
  freed a call). No trims.
- Tokens (all 5 chunks, raw JSON): prompt 105, audio 842
  (new chunks this run: prompt 46, audio 307).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.8 dB, max -1.5 dB. Focused frame:
  Column M DueDate - LOOKUP, 20-May-2025 highlighted. Result
  frame: `=TEXT('1_IT_PM_DATA'!M4,"MMM-YYYY")`, May-2025.
  Error frame: TEXT result +1 -> #VALUE! with the red banner.
  Header "Demo 15 / 42", tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 18_LEN_TRIM_UPPER.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/18_LEN_TRIM_UPPER.mp4`, 614,533 bytes, 30.91 s.
- Chunk durations (s): 7.52, 4.17, 5.20, 9.48, 4.32. Sum 30.69 s
  (difference 0.22 s, within 0.5 s). All 5 chunks generated new
  this run, flash-tts / Achird, same style. Trim: chunk 1,
  0.27 s leading silence (0.05 s pad kept). Chunks 3 and 4 each
  needed a 45 s-backoff retry (rolling refill freed a call).
- Tokens (all 5 chunks, raw JSON): prompt 105, audio 775.
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -18.8 dB, max -0.7 dB. Focused frame:
  Column A WorkID / Column F Owner - LOOKUP, IT-001 ->
  Ayesha Khan highlighted, B–E gap indicator. Result frame:
  `=PROPER(TRIM(B2))`, messy name -> Ayesha Khan. Error frame:
  MATCH on untrimmed name -> #N/A with the red banner. Header
  "Demo 18 / 42", tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 17_LEFT_MID_RIGHT — PARTIAL, 3 of 5 chunks (flash-lite-tts)

- Chunk 0 reused cached (7.36 s). Chunks 1–2 generated this
  run, lite model: 7.96 s (prompt 21, audio 199), 5.72 s
  (prompt 11, audio 143); chunk 2 needed a 45 s-backoff retry.
  Chunk 3: 3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (lite
  exhausted) -> demo left partial, no video file. Sticky
  flash-lite-tts; do NOT build with lite/flash mixing. Resume:
  chunks 3–4 ONLY, flash-lite-tts, after quota refill.

### 19_DATE_MONTH_YEAR — PARTIAL, 1 of 5 chunks (flash-tts)

- Fresh demo, flash-tts (flash not yet exhausted when reached).
  Chunk 0 generated this run: 6.32 s (prompt 20, audio 158).
  Text: "Is demo mein hum DATE, MONTH aur YEAR seekhenge. DATE
  asli date banata hai." Chunk 1: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (flash also exhausted) -> run stopped
  cleanly. No video file. The 1-chunk flash record is sticky
  and is NOT reset (unlike a 0-chunk record): resume chunks
  1–4 ONLY, flash-tts.

### 20–42 — NOT ATTEMPTED in this run

- 0 chunks, no files. Next run order: 17 chunks 3–4 (lite),
  19 chunks 1–4 (flash), then 20–42 in order (flash first per
  demo, lite only when flash is exhausted and the demo has no
  flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Lite-tts: demo 17 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Flash-tts: demo 19 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 15 chunk 4 retry 2, demo 17 chunk 2
  retry 1, demo 18 chunks 3 and 4 retry 1). No retries were
  burned beyond the builder's 3-attempt rule; the run stopped
  as soon as both models reported exhausted. Builder exit 0.
- Calls this run (from `resume_results.json`): 10 generated
  (8 flash-tts + 2 flash-lite-tts), 4 reused, 0 non-quota
  failures. New audio tokens this run: 1,582 (15: 307, 17: 342,
  18: 775, 19: 158; prompt 203).

## Drive sync (this run)

- Folder listing checked first (before uploading): 15 mp4s
  present (01–14, 16), matching local sizes. 15_TEXT.mp4 and
  18_LEN_TRIM_UPPER.mp4 were NOT present, so only they were
  uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name <file>`.
  Uploads returned Drive file IDs: 15
  `1TayDAEJCVbtabGSlANGACevEvR6X52-M`, 18
  `1HS4zv_XqMokz0U-wzYk4IdazfzGBJA_-`.
- Post-upload listing re-checked: 17 mp4s (01–16, 18) with
  sizes matching the local files exactly (15: 671,711 bytes;
  18: 614,533 bytes). Demo 17 is correctly absent (still
  partial). No file was moved, deleted, or trashed in Drive;
  backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 18 (flash) —
  17 of 42 demos.
- Partial: 17 (3/5, lite), 19 (1/5, flash). Untouched: 20–42.
  No 0-chunk sticky model record needed resetting this run.
  `demo_work_v5/progress.json` records every finished chunk and
  is the source of truth; the resume builder regenerates nothing
  already present.
- Run artifacts: `demo_work_v5/cron_run_2235.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_{15,18}/`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 10 — cron resume run 2026-11-03 00:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
17:47 UTC on 2026-11-02). Nothing was uploaded, moved, or deleted
except the single Drive upload of 17_LEFT_MID_RIGHT.mp4 recorded
below. `output/demos/`, `output/demos_v4/`, and all backup folders
were not touched. Progress backup:
`demo_work_v5/progress.json.bak-0005` (taken before the run).

## Result per demo (this run)

### 17_LEFT_MID_RIGHT.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/17_LEFT_MID_RIGHT.mp4`, 701,060 bytes, 35.12 s.
- Chunk durations (s): 7.36, 7.96, 5.72, 7.64, 6.20. Sum 34.88 s
  (difference 0.24 s, within 0.5 s). Chunks 0–2 reused cached;
  chunks 3–4 generated new this run, lite model, Achird, same
  style. Chunk 3 needed two 45 s-backoff retries (rolling refill
  freed a call). No trims.
- Tokens (all 5 chunks, raw JSON — the source of truth;
  `progress.json` `tokens` for this demo double-counts on cache
  reuse and is NOT used here): prompt 107, audio 872
  (new chunks this run: prompt 52, audio 346).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.7 dB, max -1.4 dB. Focused frame:
  Column A WorkID - LOOKUP, IT-001 highlighted, IT-001 to IT-005
  listed. Result frame: `=LEFT('1_IT_PM_DATA'!A4,2)&" | "
  &RIGHT('1_IT_PM_DATA'!A4,3)`, IT-001 -> IT | 001. Error frame:
  literal "IT-1001" -> IT | 001 (wrong) with the red banner.
  Header "Demo 17 / 42", tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 19_DATE_MONTH_YEAR — STILL PARTIAL, 1 of 5 chunks (flash-tts)

- Chunk 0 reused cached (6.32 s). Chunk 1: 3 attempts, all HTTP
  429 RESOURCE_EXHAUSTED (flash-tts) -> demo left partial, no video
  file. Sticky flash-tts; do NOT build with lite. Resume: chunks
  1–4 ONLY, flash-tts, after quota refill.

### 20_EOMONTH — PARTIAL, 1 of 5 chunks (flash-lite-tts)

- Fresh demo, reached with flash-tts already exhausted, so the
  builder chose lite for the whole demo per the per-demo rule.
  Chunk 0 generated this run, lite model: 6.68 s (prompt 26,
  audio 174), after one 45 s-backoff retry on quota; trim 0.28 s
  leading silence (0.05 s pad kept). Text: "Is demo mein hum
  EOMONTH seekhenge. EOMONTH mahine ka akhri din batata hai."
  Chunk 1: 3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (lite also
  exhausted) -> run stopped cleanly. No video file. The 1-chunk
  lite record is sticky and is NOT reset (unlike a 0-chunk
  record): resume chunks 1–4 ONLY, flash-lite-tts.

### 21–42 — NOT ATTEMPTED in this run

- 0 chunks, no files. Next run order: 19 chunks 1–4 (flash),
  20 chunks 1–4 (lite), then 21–42 in order (flash first per
  demo, lite only when flash is exhausted and the demo has no
  flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 19 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 20 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 17 chunk 3 retry 2, demo 20 chunk 0
  retry 1). No retries were burned beyond the builder's
  3-attempt rule; the run stopped as soon as both models
  reported exhausted. Builder exit 0.
- Calls this run (from `resume_results.json`): 3 generated
  (all flash-lite-tts), 4 reused, 0 non-quota failures. New
  audio tokens this run: 520 (17: 346, 20: 174; prompt 78).

## Drive sync (this run)

- Folder listing checked first: 17 mp4s present (01–16, 18),
  matching local sizes. 17_LEFT_MID_RIGHT.mp4 was NOT present,
  so only it was uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 17_LEFT_MID_RIGHT.mp4`
  Upload returned Drive file ID
  `1oFe1rEUk_qbtJ4CBXO0KVfxPdlwdx34N`.
- Post-upload listing re-checked: 18 mp4s (01–18) with sizes
  matching the local files exactly (17: 701,060 bytes). No file
  was moved, deleted, or trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash) — 18 of 42 demos.
- Partial: 19 (1/5, flash), 20 (1/5, lite). Untouched: 21–42.
  No 0-chunk sticky model record needed resetting this run.
  `demo_work_v5/progress.json` records every finished chunk and
  is the source of truth; the resume builder regenerates nothing
  already present.
- Run artifacts: `demo_work_v5/cron_run_0005.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_17/`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 11 — cron resume run 2026-11-03 01:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
19:08 UTC on 2026-11-02). Nothing was uploaded, moved, or deleted
except the single Drive upload of 19_DATE_MONTH_YEAR.mp4 recorded
below. `output/demos/`, `output/demos_v4/`, and all backup folders
were not touched. Progress backup:
`demo_work_v5/progress.json.bak-0135` (taken before the run).

## Result per demo (this run)

### 19_DATE_MONTH_YEAR.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/19_DATE_MONTH_YEAR.mp4`, 547,705 bytes, 25.81 s.
- Chunk durations (s): 6.32, 3.92, 3.72, 7.00, 4.62. Sum 25.58 s
  (difference 0.23 s, within 0.5 s). Chunk 0 reused cached;
  chunks 1–4 generated new this run, flash-tts / Achird, same
  style. Trims: chunk 3, 0.28 s; chunk 4, 0.30 s leading silence
  (0.05 s pad kept, recorded by the builder).
- Tokens (all 5 chunks, raw JSON): prompt 85, audio 654
  (new chunks this run: prompt 65, audio 496).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.7 dB, max -1.7 dB. Focused frame:
  Column M DueDate - LOOKUP, 20-May-2025 highlighted. Result
  frame: `=MONTH('1_IT_PM_DATA'!M4)`, 20-May-2025 -> 5.
  Error frame: MONTH on text "20-May-2025" -> #VALUE! with the
  red banner. Header "Demo 19 / 42", tabs fully visible,
  nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 20_EOMONTH — STILL PARTIAL, 1 of 5 chunks (flash-lite-tts)

- Chunk 0 reused cached (6.68 s). Chunk 1: 3 attempts, all HTTP
  429 RESOURCE_EXHAUSTED (lite-tts) -> demo left partial, no video
  file. Sticky flash-lite-tts; do NOT build with flash. Resume:
  chunks 1–4 ONLY, flash-lite-tts, after quota refill.

### 21_NETWORKDAYS — PARTIAL, 4 of 5 chunks (flash-tts)

- Fresh demo, flash-tts. Chunks 0–3 generated this run, each
  after 45 s-backoff retries on quota (rolling refill freed a
  call each time): 5.76 s (prompt 22, audio 144), 6.12 s
  (18, 153), 2.44 s (7, 61), 7.80 s (27, 195). Chunk 1 text
  keeps the restored "IT-001". No trims. Chunk 4: 3 attempts,
  all HTTP 429 RESOURCE_EXHAUSTED -> demo left partial, no video
  file. Sticky flash-tts; do NOT build with lite. Resume: chunk
  4 ONLY, flash-tts, after quota refill.

### 22–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 20 chunks 1–4 (lite), 21 chunk 4 (flash),
  then 22–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Lite-tts: demo 20 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Flash-tts: demo 21 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 21 chunks 0–3, each after retries). No
  retries were burned beyond the builder's 3-attempt rule; the
  run stopped as soon as both models reported exhausted.
  Builder exit 0.
- Calls this run (from `resume_results.json`): 8 generated
  (all flash-tts), 2 reused, 0 non-quota failures. New audio
  tokens this run: 1,049 (19: 496, 21: 553; prompt 139).

## Drive sync (this run)

- Folder listing checked first (01:47 PKT): 18 mp4s present
  (01–18), matching local sizes. 19_DATE_MONTH_YEAR.mp4 was NOT
  present, so only it was uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 19_DATE_MONTH_YEAR.mp4`
  Upload returned Drive file ID
  `15X6xdPcmGzzGJWIckgk4tbGpqSyrVXbC`.
- Post-upload listing re-checked: 19 mp4s (01–19) with sizes
  matching the local files exactly (19: 547,705 bytes). No file
  was moved, deleted, or trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash) — 19 of 42 demos.
- Partial: 20 (1/5, lite), 21 (4/5, flash). Untouched: 22–42.
  No 0-chunk sticky model record needed resetting this run.
  `demo_work_v5/progress.json` records every finished chunk and
  is the source of truth; the resume builder regenerates nothing
  already present.
- Run artifacts: `demo_work_v5/cron_run_0135.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_19/`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 12 — cron resume run 2026-11-03 03:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
20:47 UTC on 2026-11-02 / 01:47 PKT). Nothing was uploaded, moved,
or deleted except the single Drive upload of 21_NETWORKDAYS.mp4
recorded below. `output/demos/`, `output/demos_v4/`, and all backup
folders were not touched. Progress backup:
`demo_work_v5/progress.json.bak-0305` (taken before the run).

## Result per demo (this run)

### 21_NETWORKDAYS.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/21_NETWORKDAYS.mp4`, 588,853 bytes, 27.28 s.
- Chunk durations (s): 5.76, 6.12, 2.44, 7.80, 4.96. Sum 27.08 s
  (difference 0.20 s, within 0.5 s). Chunks 0–3 reused cached;
  chunk 4 generated new this run, flash-tts / Achird, same style,
  after two 45 s-backoff retries on quota (rolling refill freed a
  call). Chunk 1 text keeps the restored "IT-001". No trims.
- Tokens (all 5 chunks, raw JSON): prompt 96, audio 677
  (new chunk 4 this run: prompt 22, audio 124).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -18.5 dB, max -1.1 dB. Focused frame:
  Column L StartDate / Column M DueDate, 12-May-2025 ->
  20-May-2025 highlighted. Result frame:
  `=NETWORKDAYS('1_IT_PM_DATA'!L4,'1_IT_PM_DATA'!M4)`, 7 working
  days. Error frame: plain subtraction -> 8 days with the red
  banner. Header "Demo 21 / 42", tabs fully visible, nothing clipped.
  PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 20_EOMONTH — STILL PARTIAL, 1 of 5 chunks (flash-lite-tts)

- Chunk 0 reused cached (6.68 s). Chunk 1: attempt 0 failed with a
  NON-quota error (helper subprocess traceback in `http/client.py`
  / `socket.py`, `quota=False` in the builder log) and the builder
  marked the demo NOT BUILT with no further retry logged for that
  chunk; no chunk audio was generated. This was not a quota stop.
  Sticky flash-lite-tts; do NOT build with flash. Resume: chunks
  1–4 ONLY, flash-lite-tts, after quota refill; chunk 1 is retried
  on the next run.

### 22_UNIQUE — NOT ATTEMPTED (0 chunks; both quotas exhausted)

- Fresh demo, flash-tts first. Chunk 0: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (flash-tts), then the builder retried the
  whole demo with flash-lite-tts: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (lite also exhausted) -> run stopped cleanly.
  No chunk was generated. Its `progress.json` model was reset to
  None (undecided) after the run, with no chunk data lost, so the
  next run re-decides flash-first per the per-demo model order
  instead of treating the failed first call as a sticky lite
  choice. (Same treatment as demos 15/17 in earlier batches.)

### 23–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 20 chunks 1–4 (lite), then 22–42 in order
  (flash first per demo, lite only when flash is exhausted and
  the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 22 chunk 0, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 22 chunk 0 (whole-demo retry), 3 attempts
  (45 s backoffs), each HTTP 429 RESOURCE_EXHAUSTED with the same
  description.
- Rolling refill observed once: demo 21 chunk 4 succeeded on
  retry 2 after 45 s backoffs. No retries were burned beyond the
  builder's 3-attempt rule; the run stopped as soon as both models
  reported exhausted. Builder exit 0.
- Calls this run (from `resume_results.json`): 1 generated
  (flash-tts), 5 reused. New audio tokens this run: 124
  (demo 21 chunk 4; prompt 22). The demo 20 chunk 1 failure was
  a non-quota transport error, not a quota call.

## Drive sync (this run)

- Folder listing checked first: 19 mp4s present (01–19),
  matching local sizes. 21_NETWORKDAYS.mp4 was NOT present, so
  only it was uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 21_NETWORKDAYS.mp4`
  Upload returned Drive file ID
  `1Lm67WRYN-YTYDM09OvqNYugwkOXFLGis`.
- Post-upload listing re-checked: 20 mp4s (01–19, 21) with sizes
  matching the local files exactly (21: 588,853 bytes). Demo 20
  is correctly absent (still partial). No file was moved,
  deleted, or trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 21 (flash) — 20 of 42 demos.
- Partial: 20 (1/5, lite). Untouched: 22–42 (22 model reset
  to undecided, see above). `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_0305.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_21/`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 13 — cron resume run 2026-11-03 04:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
22:39 UTC on 2026-11-02). Nothing was uploaded, moved, or deleted
except the two Drive uploads recorded below. `output/demos/`,
`output/demos_v4/`, and all backup folders were not touched.
Progress backup:
`demo_work_v5/progress.json.bak-0435` (taken before the run).

## Result per demo (this run)

### 20_EOMONTH.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/20_EOMONTH.mp4`, 595,623 bytes, 31.10 s.
- Chunk durations (s): 6.68, 7.44, 4.72, 7.48, 4.56. Sum 30.88 s
  (difference 0.22 s, within 0.5 s). Chunk 0 reused cached;
  chunks 1–4 generated new this run, lite model, Achird, same
  style. No trims. The Batch 12 non-quota transport failure on
  chunk 1 did NOT recur; chunk 1 generated normally.
- Tokens (all 5 chunks, raw JSON): prompt 97, audio 779
  (new chunks this run: prompt 71, audio 605).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.3 dB, max -0.9 dB. Focused frame:
  Column M DueDate - LOOKUP, 20-May-2025 highlighted. Result
  frame: `=EOMONTH(DATE(2025,6,30),0)`, 30-Jun-2025. Error
  frame: EOMONTH on text "30-Jun-2025" -> #VALUE! with the red
  banner. Header "Demo 20 / 42", tabs fully visible, nothing
  clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 22_UNIQUE.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/22_UNIQUE.mp4`, 639,549 bytes, 28.15 s.
- Chunk durations (s): 7.68, 2.92, 5.36, 7.96, 4.01. Sum 27.93 s
  (difference 0.22 s, within 0.5 s). All 5 chunks generated new
  this run, flash-tts / Achird, same style. Trim: chunk 4,
  0.27 s leading silence (0.05 s pad kept, recorded by the builder).
- Tokens (all 5 chunks, raw JSON): prompt 94, audio 705.
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.8 dB, max -1.5 dB. Focused frame:
  Column B Project - LOOKUP, four projects repeating. Result
  frame: `=UNIQUE('1_IT_PM_DATA'!B4:B123)`, 4 unique projects
  spilled. Error frame: blocked spill cell -> #SPILL! with the
  red banner. Header "Demo 22 / 42", tabs fully visible,
  nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 23_SORT — PARTIAL, 3 of 5 chunks (flash-tts, blocked)

- Chunks 0–2 generated this run, flash-tts: 7.28 s (prompt 23,
  audio 182), 7.52 s (21, 188), 4.11 s (13, 110; trim 0.25 s).
  Chunk 3: 3 attempts, all HTTP 429 RESOURCE_EXHAUSTED ->
  demo left partial, no video file. Sticky flash-tts; do NOT
  build with lite. Resume: chunks 3–4 ONLY, flash-tts, after
  quota refill.

### 24_FILTER — PARTIAL, 4 of 5 chunks (flash-lite-tts, blocked)

- Chunks 0–3 generated this run, lite model (flash-tts was
  exhausted when demo 23 stopped; fresh demo, so lite per the
  per-demo rule): 7.42 s (prompt 26, audio 193; trim 0.30 s),
  8.12 s (13, 203), 6.60 s (19, 172; trim 0.28 s), 7.52 s
  (25, 188). Chunk 2 text keeps the restored "IT-005".
  Chunk 4: 3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (lite
  also exhausted) -> run stopped cleanly. No video file.
  Resume: chunk 4 ONLY, flash-lite-tts (sticky).

### 25–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 23 chunks 3–4 (flash), 24 chunk 4 (lite),
  then 25–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 23 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 24 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed repeatedly in this run: single calls
  freed up after 45 s backoffs (demo 23 chunk 2 retry 1, demo 24
  chunks 0–3 retry 1 each). No retries were burned beyond the
  builder's 3-attempt rule; the run stopped as soon as both
  models reported exhausted. Builder exit 0.
- Calls this run (from `resume_results.json`): 16 generated
  (8 flash-tts + 8 flash-lite-tts), 1 reused, 0 non-quota
  failures. New audio tokens this run: 2,546 (20: 605, 22: 705,
  23: 480, 24: 756; prompt 305).

## Drive sync (this run)

- Folder listing checked first (Drive `files list`, parent
  1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4): 20 mp4s present (01–19,
  21), matching local sizes. 20_EOMONTH.mp4 and 22_UNIQUE.mp4
  were NOT present, so only they were uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name <file>`.
  Uploads returned Drive file IDs: 20
  `13s2oa1kwighs3485r8B6rFsNMCFotjsd`, 22
  `1mj0_FltZU4uSIQnbSKTht8qpB7WMlW5N`.
- Post-upload listing re-checked: 22 mp4s (01–22) with sizes
  matching the local files exactly (20: 595,623 bytes;
  22: 639,549 bytes). No file was moved, deleted, or trashed
  in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash)
  — 22 of 42 demos.
- Partial: 23 (3/5, flash), 24 (4/5, lite). Untouched: 25–42.
  No 0-chunk sticky model record needed resetting this run.
  `demo_work_v5/progress.json` records every finished chunk and
  is the source of truth; the resume builder regenerates nothing
  already present.
- Run artifacts: `demo_work_v5/cron_run_0435.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_{20,22}/`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 14 — cron resume run 2026-11-03 06:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
00:44 PKT, the Batch 13 build report). Nothing was uploaded, moved,
or deleted; no new video file was produced. `output/demos/`,
`output/demos_v4/`, and all backup folders were not touched.
Progress backup:
`demo_work_v5/progress.json.bak-0605` (taken before the run).

## Result per demo (this run)

### 23_SORT — STILL PARTIAL, 3 of 5 chunks (flash-tts)

- Chunks 0–2 reused cached (7.28, 7.52, 4.11 s). Chunk 3:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (flash-tts free-tier
  requests per day exceeded). No video file. Sticky flash-tts;
  do NOT build with lite. Resume: chunks 3–4 ONLY, flash-tts,
  after quota refill.

### 24_FILTER — STILL PARTIAL, 4 of 5 chunks (flash-lite-tts)

- Chunks 0–3 reused cached (7.42, 8.12, 6.60, 7.52 s). Chunk 4:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (lite-tts also
  exhausted) -> run stopped cleanly. No video file. Sticky
  flash-lite-tts. Resume: chunk 4 ONLY, flash-lite-tts.

### 25–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order unchanged: 23 chunks 3–4 (flash), 24 chunk 4
  (lite), then 25–42 in order (flash first per demo, lite only
  when flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 23 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 24 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Unlike earlier batches, NO rolling-refill success was observed
  in this run's retries: no call freed up during the 45 s
  backoffs. No retries were burned beyond the builder's
  3-attempt rule; the run stopped as soon as both models
  reported exhausted. Builder exit 0.
- Calls this run (from `resume_results.json`): 0 generated,
  7 reused, 0 non-quota failures. New audio tokens this run: 0.

## Drive sync (this run)

- None. No demo finished this run, so no folder listing check
  or upload was needed and none was made. Drive still holds
  the same 22 mp4s (01–22 contiguous) from earlier runs.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash)
  — 22 of 42 demos.
- Partial: 23 (3/5, flash), 24 (4/5, lite). Untouched: 25–42.
  No 0-chunk sticky model record needed resetting this run.
  `demo_work_v5/progress.json` records every finished chunk and
  is the source of truth; the resume builder regenerates nothing
  already present.
- Bookkeeping drift noted (known, harmless, same as earlier
  batches): re-recording the 7 reused chunks overwrote their
  `trims` with 0.0, flipped `provenance` from "generated" to
  "cached", and double-counted `tokens` in progress.json (23:
  57/480 -> 114/960; 24: 83/756 -> 166/1512). Chunk durations,
  models, statuses, and the chunk MP3 files themselves are
  unchanged; the per-chunk `*.raw.json` files remain the token
  source of truth.
- Run artifacts: `demo_work_v5/cron_run_0605.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run). No new frame checks (no demo built).
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 15 — cron resume run 2026-11-03 07:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
01:12 PKT, the Batch 14 build-report update). Nothing was uploaded,
moved, or deleted except the two Drive uploads recorded below.
`output/demos/`, `output/demos_v4/`, and all backup folders were
not touched. Progress backup:
`demo_work_v5/progress.json.bak-0735` (taken before the run).

## Result per demo (this run)

### 23_SORT_SORTBY.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/23_SORT_SORTBY.mp4`, 678,058 bytes, 30.25 s.
- Chunk durations (s): 7.28, 7.52, 4.11, 6.44, 4.64. Sum 29.99 s
  (difference 0.26 s, within 0.5 s). Chunks 0–2 reused cached;
  chunks 3–4 generated new this run, flash-tts / Achird, same
  style. Chunk 3 needed one 45 s-backoff retry (rolling refill
  freed a call). Trim: chunk 3, 0.32 s leading silence (0.05 s
  pad kept, recorded by the builder).
- Tokens (all 5 chunks, raw JSON — the source of truth;
  `progress.json` `tokens` for this demo double-counts on cache
  reuse and is NOT used here): prompt 96, audio 765
  (new chunks this run: prompt 39, audio 285).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.0 dB, max -1.1 dB. Focused frame:
  Column A WorkID / Column K Variance - RESULT, IT-001 -> -1.9,
  IT-005 -> 5.0. Result frame: `=SORT('1_IT_PM_DATA'!A4:K123,11,-1)`,
  largest overrun first, IT-110 HRMS Upgrade -> 7.5. Error frame
  (verbatim v4 visual): wrong order `,11,1)`, smallest first,
  IT-004 -> -19.5, with the red banner. Header "Demo 23 / 42",
  tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 24_FILTER.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/24_FILTER.mp4`, 714,465 bytes, 33.94 s.
- Chunk durations (s): 7.42, 8.12, 6.60, 7.52, 4.04. Sum 33.70 s
  (difference 0.24 s, within 0.5 s). Chunks 0–3 reused cached;
  chunk 4 generated new this run, lite model, Achird, same
  style, after one 45 s-backoff retry (rolling refill freed a
  call). Chunk 2 text keeps the restored "IT-005". No trims.
- Tokens (all 5 chunks, raw JSON — the source of truth;
  `progress.json` `tokens` for this demo double-counts on cache
  reuse and is NOT used here): prompt 96, audio 857
  (new chunk this run: prompt 13, audio 101).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.7 dB, max -1.4 dB. Focused frame:
  Column A WorkID / Column B Project, Column G Status /
  Column H Priority - LOOKUP, Delayed + Critical highlighted.
  Result frame: `=FILTER('1_IT_PM_DATA'!A4:P123,...,"None found")`,
  5 Delayed + Critical rows, IT-005 first. Error frame: FILTER
  without if-empty -> #CALC! with the red banner and the
  "None found" fix. Header "Demo 24 / 42", tabs fully visible,
  nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 25_MAX_MIN_LARGE_SMALL — NOT ATTEMPTED (0 chunks; both quotas exhausted)

- Fresh demo, flash-tts first. Chunk 0: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (flash-tts), then the builder retried the
  whole demo with flash-lite-tts: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (lite also exhausted) -> run stopped cleanly.
  No chunk was generated. Its `progress.json` model was reset to
  None (undecided) after the run, with no chunk data lost, so the
  next run re-decides flash-first per the per-demo model order
  instead of treating the failed first call as a sticky lite
  choice. (Same treatment as demos 15/17/22 in earlier batches.)

### 26–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 25–42 in order (flash first per demo, lite
  only when flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 25 chunk 0, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 25 chunk 0 (whole-demo retry), 3 attempts
  (45 s backoffs), each HTTP 429 RESOURCE_EXHAUSTED with the same
  description.
- Rolling refill observed in this run before the stop: single
  calls freed up after 45 s backoffs (demo 23 chunk 3 retry 1,
  demo 24 chunk 4 retry 1). No retries were burned beyond the
  builder's 3-attempt rule; the run stopped as soon as both
  models reported exhausted. Builder exit 0.
- Calls this run (from `resume_results.json`): 3 generated
  (2 flash-tts + 1 flash-lite-tts), 7 reused, 0 non-quota
  failures. New audio tokens this run: 386 (23: 285, 24: 101;
  prompt 52).

## Drive sync (this run)

- Folder listing checked first (Drive `files list`, query
  `'1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4' in parents and
  trashed=false`, 07:42 PKT): 22 mp4s present (01–22),
  matching local sizes. 23_SORT_SORTBY.mp4 and 24_FILTER.mp4
  were NOT present, so only they were uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name <file>`.
  Uploads returned Drive file IDs: 23
  `12kyfmzJyvFD7RaJTtXMpxVe5NgztjtBL`, 24
  `1S4MLO8V7UzfwpSNG602Bf71C79XEHrXS`.
- Post-upload listing re-checked: 24 mp4s (01–24) with sizes
  matching the local files exactly (23: 678,058 bytes;
  24: 714,465 bytes). No file was moved, deleted, or trashed
  in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite) — 24 of 42 demos.
- Partial: none. Untouched: 25–42 (25 model reset to
  undecided, see above). `demo_work_v5/progress.json` records
  every finished chunk and is the source of truth; the resume
  builder regenerates nothing already present.
- Bookkeeping drift noted (known, harmless, same as earlier
  batches): re-recording the 7 reused chunks overwrote their
  `trims` with 0.0, flipped `provenance` from "generated" to
  "cached", and double-counted `tokens` in progress.json (23:
  96/765 -> 210/1725; 24: 96/857 -> 262/2369). Chunk durations,
  models, statuses, and the chunk MP3 files themselves are
  unchanged; the per-chunk `*.raw.json` files remain the token
  source of truth.
- Run artifacts: `demo_work_v5/cron_run_0735.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_{23,24}/`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 16 — cron resume run 2026-11-03 09:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
02:43 UTC / 07:43 PKT, the Batch 15 build report). Nothing was
uploaded, moved, or deleted except the single Drive upload of
26_RANK.mp4 recorded below. `output/demos/`, `output/demos_v4/`,
and all backup folders were not touched. Progress backup:
`demo_work_v5/progress.json.bak-0905` (taken before the run).

## Result per demo (this run)

### 26_RANK.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/26_RANK.mp4`, 701,923 bytes, 34.59 s.
- Chunk durations (s): 5.72, 6.61, 9.02, 8.13, 4.88. Sum 34.36 s
  (difference 0.23 s, within 0.5 s). All 5 chunks generated new
  this run, lite model, Achird, same style. Trims: chunk 1,
  0.27 s; chunk 2, 0.26 s; chunk 3, 0.27 s leading silence
  (0.05 s pad kept, recorded by the builder). Chunk 0 needed a
  45 s-backoff retry and chunk 1 two 45 s-backoff retries
  (rolling refill freed a call each time).
- Tokens (all 5 chunks, raw JSON): prompt 106, audio 879.
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -20.1 dB, max -1.4 dB. Focused frame:
  Column B Project / Column G Status - LOOKUP, Delayed
  highlighted, C–F gap indicator. Result frame:
  `=RANK(B2,$B$2:$B$5,0)`, HRMS Upgrade 6 -> rank 1.
  Error frame: unlocked `=RANK(B2,B2:B5,0)` -> wrong ranks
  (Inventory API 2, Service Desk 1) with the red banner and the
  `$B$2:$B$5` fix. Header "Demo 26 / 42", tabs fully visible,
  nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 25_MAX_MIN_LARGE_SMALL — PARTIAL, 1 of 5 chunks (flash-tts)

- Chunk 0 generated this run, flash-tts: 8.76 s (prompt 26,
  audio 219). Text: "Is demo mein hum MAX, MIN, LARGE aur SMALL
  seekhenge. LARGE, k-th bara value batata hai." Chunk 1:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (flash-tts) ->
  demo left partial, no video file. Sticky flash-tts; do NOT
  build with lite. Resume: chunks 1–4 ONLY, flash-tts, after
  quota refill.

### 27_PERCENTILE — PARTIAL, 2 of 5 chunks (flash-lite-tts)

- Chunks 0–1 generated this run, lite model (flash-tts was
  exhausted when demo 25 stopped; fresh demo, so lite per the
  per-demo rule): 7.52 s (prompt 26, audio 188), 4.76 s
  (prompt 12, audio 119). Chunk 0 needed a 45 s-backoff retry
  (rolling refill freed a call). Texts: "Is demo mein hum
  PERCENTILE seekhenge. PERCENTILE batata hai ke kitne percent
  values is se neeche hain." / "Formula mein Actual Hours, phir
  zero point nine five." Chunk 2: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (lite also exhausted) -> run stopped
  cleanly. No video file. Resume: chunks 2–4 ONLY,
  flash-lite-tts (sticky).

### 28–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 25 chunks 1–4 (flash), 27 chunks 2–4
  (lite), then 28–42 in order (flash first per demo, lite only
  when flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 25 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 27 chunk 2, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 26 chunks 0 and 1, demo 27 chunk 0). No
  retries were burned beyond the builder's 3-attempt rule; the
  run stopped as soon as both models reported exhausted.
  Builder exit 0.
- Calls this run (from `resume_results.json`): 8 generated
  (1 flash-tts + 7 flash-lite-tts), 0 reused, 0 non-quota
  failures. New audio tokens this run: 1,405 (25: 219, 26: 879,
  27: 307; prompt 170).

## Drive sync (this run)

- Folder listing checked first (Drive `files list`, query
  `'1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4' in parents and
  trashed=false`): 24 mp4s present (01–24), matching local
  sizes. 26_RANK.mp4 was NOT present, so only it was uploaded,
  with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 26_RANK.mp4`
  Upload returned Drive file ID
  `1kOcmVm4ThfNU_XgThV4kE6K_Qtyl3rfm`.
- Post-upload listing re-checked: 25 mp4s (01–24, 26) with
  sizes matching the local files exactly (26: 701,923 bytes).
  Demo 25 is correctly absent (still partial). No file was
  moved, deleted, or trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 26 (lite) — 25 of 42 demos.
- Partial: 25 (1/5, flash), 27 (2/5, lite). Untouched: 28–42.
  No 0-chunk sticky model record needed resetting this run.
  `demo_work_v5/progress.json` records every finished chunk and
  is the source of truth; the resume builder regenerates nothing
  already present. No bookkeeping drift this run (0 reused chunks).
- Run artifacts: `demo_work_v5/cron_run_0905.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_26/`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 17 — cron resume run 2026-11-03 10:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
09:17 PKT, the Batch 16 build-report update). Nothing was uploaded,
moved, or deleted; no new video file was produced. `output/demos/`,
`output/demos_v4/`, and all backup folders were not touched.
Progress backup:
`demo_work_v5/progress.json.bak-1035` (taken before the run).

## Result per demo (this run)

### 25_MAX_MIN_LARGE_SMALL — PARTIAL, 3 of 5 chunks (flash-tts)

- Chunk 0 reused cached (8.76 s). Chunks 1–2 generated this run,
  flash-tts / Achird, same style: chunk 1, 6.40 s (prompt 14,
  audio 160), text "Formula mein Variance, phir teen, matlab
  teesra bara overrun."; chunk 2, 2.70 s (prompt 9, audio 74),
  text "Result, teesra bara variance hai.", after two 45 s-backoff
  retries on quota (rolling refill freed a call). Trim: chunk 2,
  0.26 s leading silence (0.05 s pad kept, recorded by the
  builder). Chunk 3: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED -> demo left partial, no video file.
  Sticky flash-tts; do NOT build with lite. Resume: chunks 3–4
  ONLY, flash-tts, after quota refill.

### 27_PERCENTILE — PARTIAL, 3 of 5 chunks (flash-lite-tts)

- Chunks 0–1 reused cached (7.52 s, 4.76 s). Chunk 2 generated
  this run, lite model, Achird, same style: 5.64 s (prompt 20,
  audio 141), text "Result, ninety fifth percentile hai, jo
  S L A upper bound ke liye use hota hai." Chunk 3: 3 attempts,
  all HTTP 429 RESOURCE_EXHAUSTED (lite also exhausted) -> run
  stopped cleanly. No video file. Sticky flash-lite-tts.
  Resume: chunks 3–4 ONLY, flash-lite-tts, after quota refill.

### 28–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 25 chunks 3–4 (flash), 27 chunks 3–4 (lite),
  then 28–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 25 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 27 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed once: demo 25 chunk 2 succeeded on
  retry 2 after 45 s backoffs. No retries were burned beyond the
  builder's 3-attempt rule; the run stopped as soon as both models
  reported exhausted. Builder exit 0.
- Calls this run (from `resume_results.json`): 3 generated
  (2 flash-tts + 1 flash-lite-tts), 3 reused, 0 non-quota
  failures. New audio tokens this run: 375 (25: 234, 27: 141;
  prompt 43), from the per-chunk `*.raw.json` files.

## Drive sync (this run)

- None. No demo finished this run, so no folder listing check
  or upload was needed and none was made. Drive still holds
  the same 25 mp4s (01–24, 26) from earlier runs.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 26 (lite) — 25 of 42 demos.
- Partial: 25 (3/5, flash), 27 (3/5, lite). Untouched: 28–42.
  No 0-chunk sticky model record needed resetting this run.
  `demo_work_v5/progress.json` records every finished chunk and
  is the source of truth; the resume builder regenerates nothing
  already present.
- Bookkeeping drift noted (known, harmless, same as earlier
  batches): re-recording the reused chunks double-counted
  `tokens` in progress.json (25: true raw totals 49/453 ->
  recorded 75/672; 27: true raw totals 58/448 -> recorded
  96/755). Chunk durations, models, statuses, and the chunk MP3
  files themselves are unchanged; the per-chunk `*.raw.json`
  files remain the token source of truth.
- Run artifacts: `demo_work_v5/cron_run_1035.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run). No new frame checks (no demo built).
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 18 — cron resume run 2026-11-03 12:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
10:42 PKT, the Batch 17 build-report update). Nothing was uploaded,
moved, or deleted except the two Drive uploads recorded below.
`output/demos/`, `output/demos_v4/`, and all backup folders were
not touched. Progress backup:
`demo_work_v5/progress.json.bak-1205` (taken before the run).

## Result per demo (this run)

### 25_MAX_MIN_LARGE_SMALL.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/25_MAX_MIN_LARGE_SMALL.mp4`, 628,591 bytes,
  30.52 s.
- Chunk durations (s): 8.76, 6.40, 2.70, 7.95, 4.48. Sum 30.29 s
  (difference 0.23 s, within 0.5 s). Chunks 0–2 reused cached;
  chunks 3–4 generated new this run, flash-tts / Achird, same
  style. Trim: chunk 3, 0.29 s leading silence (0.05 s pad kept,
  recorded by the builder). Chunk 4 needed one 45 s-backoff retry
  (rolling refill freed a call).
- Tokens (all 5 chunks, raw JSON — the source of truth;
  `progress.json` `tokens` for this demo double-counts on cache
  reuse and is NOT used here): prompt 89, audio 772
  (new chunks this run: prompt 40, audio 319).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.4 dB, max -1.5 dB. Focused frame:
  Column A WorkID / Column K Variance - RESULT, IT-001 -> -1.9,
  IT-005 -> 5. Result frame: `=LARGE('1_IT_PM_DATA'!K4:K123,3)`,
  3rd largest -> 5.9 hours (MAX 7.5, MIN -19.5). Error frame:
  LARGE 200th -> #NUM! with the red banner and the COUNT fix.
  Header "Demo 25 / 42", tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 27_PERCENTILE.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/27_PERCENTILE.mp4`, 683,660 bytes, 34.36 s.
- Chunk durations (s): 7.52, 4.76, 5.64, 9.92, 6.28. Sum 34.12 s
  (difference 0.24 s, within 0.5 s). Chunks 0–2 reused cached;
  chunks 3–4 generated new this run, lite model, Achird, same
  style. Chunk 3 needed one 45 s-backoff retry (rolling refill
  freed a call). No trims.
- Tokens (all 5 chunks, raw JSON — the source of truth;
  `progress.json` `tokens` for this demo double-counts on cache
  reuse and is NOT used here): prompt 108, audio 853
  (new chunks this run: prompt 50, audio 405).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -20.1 dB, max -0.4 dB. Focused frame:
  Column A WorkID / Column J Actual Hours - RESULT, IT-001 ->
  18.1, IT-005 -> 20.5. Result frame:
  `=PERCENTILE('1_IT_PM_DATA'!J4:J123,0.95)`, 95th percentile
  -> 20.51 hours. Error frame: k = 95 (wrong) -> #NUM! with
  the red banner and the 0.95 fix. Header "Demo 27 / 42", tabs
  fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 28 — PARTIAL, 1 of 5 chunks (flash-lite-tts, blocked)

- Fresh demo, reached with flash-tts already exhausted, so the
  builder tried flash-tts first (3 attempts, all HTTP 429),
  then retried the whole demo with lite per the per-demo rule.
  Chunk 0 generated this run, lite model: 7.16 s (prompt 20,
  audio 179). Text: "Is demo mein hum Relative aur Absolute
  reference seekhenge. Dollar sign lock ka kaam karta hai."
  Chunk 1: 3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (lite
  also exhausted) -> run stopped cleanly. No video file. The
  1-chunk lite record is sticky and is NOT reset (unlike a
  0-chunk record): resume chunks 1–4 ONLY, flash-lite-tts.

### 29–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 28 chunks 1–4 (lite), then 29–42 in order
  (flash first per demo, lite only when flash is exhausted and
  the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 28 chunk 0, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 28 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 25 chunk 4 retry 1, demo 27 chunk 3
  retry 1, and the lite fallback call for demo 28 chunk 0). No
  retries were burned beyond the builder's 3-attempt rule; the
  run stopped as soon as both models reported exhausted.
  Builder exit 0.
- Calls this run (from `resume_results.json`): 5 generated
  (2 flash-tts + 3 flash-lite-tts), 6 reused, 0 non-quota
  failures. New audio tokens this run: 903 (25: 319, 27: 405,
  28: 179; prompt 110).

## Drive sync (this run)

- Folder listing checked first (Drive `files list`, query
  `'1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4' in parents and
  trashed=false`): 25 mp4s present (01–24, 26), matching local
  sizes. 25_MAX_MIN_LARGE_SMALL.mp4 and 27_PERCENTILE.mp4 were
  NOT present, so only they were uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name <file>`.
  Uploads returned Drive file IDs: 25
  `1xFMnQSNPCjTKwS2OwVNzJu16HdfWBHKA`, 27
  `1Zp4r-TtvY2CC0fbdLW7wBpsl9EmYo4Aa`.
- Post-upload listing re-checked: 27 mp4s (01–27) with sizes
  matching the local files exactly (25: 628,591 bytes;
  27: 683,660 bytes). No file was moved, deleted, or trashed
  in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite)
  — 27 of 42 demos.
- Partial: 28 (1/5, lite). Untouched: 29–42 (all model None,
  undecided). No 0-chunk sticky model record needed resetting
  this run. `demo_work_v5/progress.json` records every finished
  chunk and is the source of truth; the resume builder
  regenerates nothing already present.
- Bookkeeping drift noted (known, harmless, same as earlier
  batches): re-recording the 6 reused chunks double-counted
  `tokens` in progress.json (25: true raw totals 89/772 ->
  recorded 164/1444; 27: true raw totals 108/853 -> recorded
  204/1608). Chunk durations, models, statuses, and the chunk
  MP3 files themselves are unchanged; the per-chunk `*.raw.json`
  files remain the token source of truth.
- Run artifacts: `demo_work_v5/cron_run_1205.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_{25,27}/`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 19 — cron resume run 2026-11-03 13:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
12:15 PKT / 07:15 UTC, the Batch 18 build-report update; `find
-mmin -20` empty). Nothing was uploaded, moved, or deleted except
the four Drive uploads recorded below. `output/demos/`,
`output/demos_v4/`, and all backup folders were not touched.
Progress backup:
`demo_work_v5/progress.json.bak-1335` (taken before the run).

## Result per demo (this run)

All 4 newly finished demos were independently re-verified by the
coordinator after the builder run (ffprobe + silencedetect +
volumedetect + frames viewed as montages in
`demo_work_v5/verify_montage_{28,29,30,32}.png`). Token totals
below are sums of the per-chunk `*.raw.json` usageMetadata (the
source of truth); `progress.json` `tokens` fields double-count
on cache reuse and are NOT used here.

### 28_REFERENCES.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/28_REFERENCES.mp4`, 792,319 bytes, 39.08 s.
- Chunk durations (s): 7.16, 8.88, 5.81, 11.76, 5.24. Sum 38.85 s
  (difference 0.23 s, within 0.5 s). Chunk 0 reused cached;
  chunks 1–4 generated new this run, lite model, Achird, same
  style. Trim: chunk 2, 0.27 s leading silence (0.05 s pad kept).
- Tokens (all 5 chunks, raw JSON): prompt 116, audio 979
  (new chunks this run: prompt 96, audio 800).
- Verification: ffprobe h264 1280x720 + AAC; silencedetect
  (noise=-40dB, d=2.0) no gap over 2 s; volumedetect mean -19.7 dB,
  max -1.6 dB. Focused frame: Column A WorkID / Column I Planned
  Hrs, rate cell $B$3 = 1,500 PKR. Result frame: `=$B6*$B$3`,
  copy down still 26,700 PKR. Error frame: relative copy
  `=B7*B4` -> 0 with the red banner. Header "Demo 28 / 42",
  tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 29_PIVOT.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/29_PIVOT.mp4`, 668,054 bytes, 32.27 s.
- Chunk durations (s): 6.84, 6.44, 5.92, 8.72, 4.12. Sum 32.04 s
  (difference 0.23 s, within 0.5 s). All 5 chunks generated new
  this run, flash-tts / Achird, same style. No trims.
- Tokens (raw JSON): prompt 114, audio 802.
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -18.4 dB, max -1.4 dB. Focused: Project / Sprint /
  Actual Hours columns. Result: COUNTIFS matrix, every project
  total 30, every Sprint total 20, grand total 120. Error:
  new project row missing from the matrix with the red banner
  and the UNIQUE-labels fix. Header "Demo 29 / 42", tabs visible.
  PASS.
- Drive: uploaded this run (see below, after a 502 retry);
  not previously in the folder.

### 30_DATA_VALIDATION.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/30_DATA_VALIDATION.mp4`, 681,519 bytes, 33.56 s.
- Chunk durations (s): 6.26, 6.40, 5.72, 10.92, 4.04. Sum 33.34 s
  (difference 0.22 s, within 0.5 s). All 5 chunks generated new
  this run, flash-tts. Trim: chunk 0, 0.26 s leading silence
  (0.05 s pad kept).
- Tokens (raw JSON): prompt 119, audio 840.
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -19.1 dB, max -1.8 dB. Focused: Status column and
  allowed list (Not Started / In Progress / Completed / Delayed /
  Blocked / On Hold). Result: "Completed" dropdown -> Accepted.
  Error: "done" -> Rejected with the "Choose a Status" error
  alert and red banner. Header "Demo 30 / 42", tabs visible. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 32_POWER_QUERY.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/32_POWER_QUERY.mp4`, 849,701 bytes, 36.88 s.
- Chunk durations (s): 6.84, 8.32, 7.20, 8.40, 5.92. Sum 36.68 s
  (difference 0.20 s, within 0.5 s). All 5 chunks generated new
  this run, lite model. No trims.
- Tokens (raw JSON): prompt 108, audio 917.
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -19.5 dB, max -1.4 dB. Focused: Project / Status /
  Actual Hours columns. Result: 5-step flow (Get Data > From
  Folder > Append > Transform > Close & Load), monthly files in
  one master table. Error frame (verbatim v4 visual, honestly
  labelled "STEPS / FLOW DEMO — no live files"): file renamed ->
  Refresh FAILED with the red banner. Header "Demo 32 / 42",
  tabs visible. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 31 — PARTIAL, 3 of 5 chunks (flash-tts, blocked)

- Chunks 0–2 generated this run, flash-tts: 6.39 s (prompt 22,
  audio 166; trim 0.25 s), 7.76 s (19, 194, after one 45 s-backoff
  retry), 6.84 s (22, 171). Chunk 3: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED -> demo left partial, no video file.
  Sticky flash-tts; do NOT build with lite. The pending chunk 3
  text file is present in `demo_work_v5/` ("Agar colour haath se
  lagaya ho, to value badalne par colour nahi badlega. Hal, rule
  use karein, haath se colour nahi."). Resume: chunks 3–4 ONLY,
  flash-tts, after quota refill.

### 33 — PARTIAL, 2 of 5 chunks (flash-lite-tts, blocked)

- Chunks 0–1 generated this run, lite model (flash-tts was
  exhausted when demo 31 stopped; fresh demo, so lite per the
  per-demo rule): 6.66 s (prompt 21, audio 173; trim 0.26 s),
  6.56 s (12, 164). Chunk 2: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (lite also exhausted) -> run stopped
  cleanly. No video file. The pending chunk 2 text file is
  present ("Training mein hum ne tees June fix rakha hai, taake
  result repeat ho. Live use mein TODAY use karein."). Resume:
  chunks 2–4 ONLY, flash-lite-tts (sticky).

### 34–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 31 chunks 3–4 (flash), 33 chunks 2–4 (lite),
  then 34–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 31 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier." The builder continued to the
  next demo (lite still available), built demo 32 fully with
  lite, then stopped at demo 33.
- Lite-tts: demo 33 chunk 2, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed once: demo 31 chunk 1 succeeded on
  retry 1 after a 45 s backoff. No retries were burned beyond
  the builder's 3-attempt rule; the run stopped as soon as both
  models reported exhausted. Builder exit 0.
- Calls this run (from `resume_results.json` / builder log):
  24 generated (13 flash-tts + 11 flash-lite-tts), 1 reused,
  0 non-quota failures. New audio tokens this run: 4,227
  (28: 800, 29: 802, 30: 840, 31: 531, 32: 917, 33: 337;
  prompt 533).

## Drive sync (this run)

- Folder listing checked first (Drive `files list`, query
  `'1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4' in parents and
  trashed=false`, 13:47 PKT): 27 mp4s present (01–27),
  matching local sizes. 28, 29, 30, 32 were NOT present, so
  only they were uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name <file>`.
- Demo 29's first upload attempt returned HTTP 502 (Google
  server error page). The folder was re-checked before any
  retry: 29_PIVOT.mp4 was absent, so the 502 had not landed
  and exactly one retry was made. The retry succeeded with a
  new Drive file ID (no duplicate was created).
- Uploads returned Drive file IDs: 28
  `1Ei3zJhqKmucBQjq6x6FJTNBNnPjSUXyY`, 29 (retry)
  `1BMMXjKrvRe6imcgmc92-rkECAlGk0zhq`, 30
  `15IxOXz-kfzmDm2EPfSmQ3wPc-VDkLyLz`, 32
  `1p93GLRezQ9ASeEfywH_EXPpJclURkN_S`.
- Post-upload listing re-checked: 31 mp4s (01–30, 32) with
  sizes matching the local files exactly (28: 792,319 bytes;
  29: 668,054; 30: 681,519; 32: 849,701). Demo 31 is correctly
  absent (still partial). No file was moved, deleted, or
  trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 32 (lite) — 31 of 42 demos.
- Partial: 31 (3/5, flash), 33 (2/5, lite). Untouched: 34–42
  (all model None, undecided). No 0-chunk sticky model record
  needed resetting this run. `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Bookkeeping drift noted (known, harmless, same as earlier
  batches): re-recording the reused demo 28 chunk 0
  double-counted `tokens` in progress.json (true raw totals
  116/979 -> recorded 136/1158) and flipped its provenance to
  "cached". Chunk durations, models, status, and the chunk MP3
  file itself are unchanged; the per-chunk `*.raw.json` files
  remain the token source of truth.
- Run artifacts: `demo_work_v5/cron_run_1335.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_{28,29,30,32}/` and montages
  `demo_work_v5/verify_montage_{28,29,30,32}.png`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 20 — cron resume run 2026-11-03 15:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
13:50 PKT, the Batch 19 build-report update; `find -mmin -20`
empty). Nothing was uploaded, moved, or deleted except the three
Drive uploads recorded below. `output/demos/`,
`output/demos_v4/`, and all backup folders were not touched.
Progress backup:
`demo_work_v5/progress.json.bak-1505` (taken before the run).

## Result per demo (this run)

All 3 newly finished demos were independently re-verified by the
coordinator after the builder run (ffprobe + silencedetect +
volumedetect + focused/result/error frames viewed, montages in
`demo_work_v5/verify_montage_{31,33,34}.png`). Token totals below
are sums of the per-chunk `chunk_*.raw.json` usageMetadata (the
source of truth); `progress.json` `tokens` fields double-count
on cache reuse and are NOT used here.

### 31_CONDITIONAL_FORMATTING.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/31_CONDITIONAL_FORMATTING.mp4`, 733,318 bytes,
  34.57 s.
- Chunk durations (s): 6.39, 7.76, 6.84, 8.64, 4.71. Sum 34.34 s
  (difference 0.23 s, within 0.5 s). Chunks 0–2 reused cached;
  chunks 3–4 generated new this run, flash-tts / Achird, same
  style. Chunk 3 needed two 45 s-backoff retries (rolling refill
  freed a call). Trims: chunk 3, 0.28 s; chunk 4, 0.33 s leading
  silence (0.05 s pad kept, recorded by the builder).
- Tokens (all 5 chunks, raw JSON): prompt 113, audio 880
  (new chunks this run: prompt 50, audio 349).
- Verification: ffprobe h264 1280x720 + AAC; silencedetect
  (noise=-40dB, d=2.0) no gap over 2 s; volumedetect mean -19.3 dB,
  max -1.3 dB. Focused frame: Column A WorkID / Column G Status /
  Column K Variance, Delayed peach, Completed green, variance
  green-yellow-red. Result frame: rules table (Delayed -> red fill,
  Completed -> green fill, Variance 3-colour scale). Error frame:
  hand colouring -> colour does not change, red banner, rule fix.
  Header "Demo 31 / 42", tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 33_TODAY.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/33_TODAY.mp4`, 667,015 bytes, 33.33 s.
- Chunk durations (s): 6.66, 6.56, 8.03, 7.48, 4.36. Sum 33.09 s
  (difference 0.24 s, within 0.5 s). Chunks 0–1 reused cached;
  chunks 2–4 generated new this run, lite model, Achird, same
  style. Chunk 3 needed one 45 s-backoff retry (rolling refill
  freed a call). Trim: chunk 2, 0.29 s leading silence (0.05 s
  pad kept).
- Tokens (all 5 chunks, raw JSON): prompt 99, audio 842
  (new chunks this run: prompt 66, audio 505).
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -18.9 dB, max -0.1 dB. Focused: Column A WorkID /
  Column G Status / Column M DueDate, 20-May-2025 highlighted.
  Result: `=TODAY()`, training date 30-Jun-2025, IT-002 ->
  51 days overdue. Error: report changes daily, 51 today
  (50 yesterday), red banner, paste-as-value fix. Header
  "Demo 33 / 42", tabs visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 34_WORKDAY.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/34_WORKDAY.mp4`, 692,404 bytes, 32.91 s.
- Chunk durations (s): 6.34, 6.20, 6.04, 8.12, 5.96. Sum 32.66 s
  (difference 0.25 s, within 0.5 s). All 5 chunks generated new
  this run, lite model (flash-tts was exhausted on chunk 0, so
  the whole demo retried with lite per the per-demo rule),
  Achird, same style. Chunk 1 and chunk 2 each needed a 45 s-backoff
  retry (rolling refill freed a call). Trim: chunk 0, 0.30 s
  leading silence (0.05 s pad kept).
- Tokens (raw JSON): prompt 103, audio 824.
- Verification: ffprobe h264 1280x720 + AAC; no silence gap over
  2 s; mean -19.4 dB, max -1.8 dB. Focused: Column A WorkID /
  Column M DueDate / Next working day, 20-May-2025 -> 21-May-2025.
  Result: `=WORKDAY('1_IT_PM_DATA'!M4,1)`, 21-May-2025.
  Error/fix: no holidays list -> date may be off, holidays-list
  fix. Header "Demo 34 / 42", tabs visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 35_EDATE — PARTIAL, 2 of 5 chunks (flash-lite-tts, blocked)

- Chunks 0–1 generated this run, lite model (flash-tts was
  exhausted when demo 34 finished; fresh demo, so lite per the
  per-demo rule): 7.09 s (prompt 26, audio 184; trim 0.27 s),
  5.48 s (prompt 16, audio 137). Chunk 2: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (lite also exhausted) -> run stopped
  cleanly. No video file. The pending chunk 2 text file is
  present in `demo_work_v5/` ("Result, bees June, do hazar
  pachees."). The 2-chunk lite record is sticky and is NOT
  reset (unlike a 0-chunk record): resume chunks 2–4 ONLY,
  flash-lite-tts, after quota refill.

### 36–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 35 chunks 2–4 (lite), then 36–42 in order
  (flash first per demo, lite only when flash is exhausted and
  the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 34 chunk 0, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier." (Whole-demo lite retry followed.)
- Lite-tts: demo 35 chunk 2, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed repeatedly in this run: single calls
  freed up after 45 s backoffs (demo 31 chunk 4 retry 2, demo 33
  chunk 3 retry 1, demo 34 chunks 1 and 2 retry 1, demo 35
  chunks 0 and 1 after retries). No retries were burned beyond
  the builder's 3-attempt rule; the run stopped as soon as both
  models reported exhausted. Builder exit 0.
- Calls this run (from `resume_results.json`): 12 generated
  (2 flash-tts + 10 flash-lite-tts), 5 reused, 0 non-quota
  failures. New audio tokens this run: 1,999 (31: 349, 33: 505,
  34: 824, 35: 321; prompt 261).

## Drive sync (this run)

- Folder listing checked first (Drive `files list`, query
  `'1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4' in parents and
  trashed=false`, 15:21 PKT): 31 mp4s present (01–30, 32),
  matching local sizes. 31, 33, 34 were NOT present, so only
  they were uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name <file>`.
  Uploads returned Drive file IDs: 31
  `10TcZ6znKXHVtevUoHudoKHqSetSGeM2x`, 33
  `1O-3jtK8JjTWjKOp_tNEDPQ56l7ttr45c`, 34
  `1BjQklkj1m2x_ZOVZwONiZfIZBCiDL0AB`.
- Post-upload listing re-checked: 34 mp4s (01–34 contiguous)
  with sizes matching the local files exactly (31: 733,318 bytes;
  33: 667,015; 34: 692,404). No file was moved, deleted, or
  trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 31 (flash), 32 (lite),
  33 (lite), 34 (lite) — 34 of 42 demos.
- Partial: 35 (2/5, lite). Untouched: 36–42 (all model None,
  undecided). No 0-chunk sticky model record needed resetting
  this run. `demo_work_v5/progress.json` records every finished
  chunk and is the source of truth; the resume builder
  regenerates nothing already present.
- Bookkeeping drift noted (known, harmless, same as earlier
  batches): re-recording the 5 reused chunks double-counted
  `tokens` in progress.json (31: true raw totals 113/880 ->
  recorded 176/1411; 33: true raw totals 99/842 -> recorded
  132/1179) and flipped their provenance to "cached". Chunk
  durations, models, statuses, and the chunk MP3 files
  themselves are unchanged; the per-chunk `*.raw.json` files
  remain the token source of truth.
- Run artifacts: `demo_work_v5/cron_run_1505.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_{31,33,34}/` and montages
  `demo_work_v5/verify_montage_{31,33,34}.png`,
  `demo_work_v5/drive_listing_1505.txt`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 21 — cron resume run 2026-11-03 16:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
15:23 PKT, the Batch 20 build-report update; `find -mmin -20`
empty). Nothing was uploaded, moved, or deleted; no new video
file was produced. `output/demos/`, `output/demos_v4/`, and all
backup folders were not touched. Progress backup:
`demo_work_v5/progress.json.bak-1635` (taken before the run).

## Result per demo (this run)

### 35_EDATE — STILL PARTIAL, 2 of 5 chunks (flash-lite-tts)

- Chunks 0–1 reused cached (7.09 s, 5.48 s). Chunk 2: 3 attempts,
  all HTTP 429 RESOURCE_EXHAUSTED (lite-tts free-tier requests per
  day exceeded). No video file. Sticky flash-lite-tts; do NOT
  build with flash. Resume: chunks 2–4 ONLY, flash-lite-tts,
  after quota refill. Pending chunk 2 text: "Result, bees June,
  do hazar pachees."

### 36_DATEDIF — PARTIAL, 2 of 5 chunks (flash-tts, blocked)

- Chunks 0–1 generated this run, flash-tts / Achird, same style:
  chunk 0, 8.49 s (prompt 31, audio 221; trim 0.31 s leading
  silence, 0.05 s pad kept); chunk 1, 5.24 s (prompt 15, audio
  131). Texts keep the correct spelling DATEDIF. Chunk 2:
  3 attempts, all ending in quota exhaustion (attempts 0 and 2
  HTTP 429 RESOURCE_EXHAUSTED; attempt 1 a non-quota transport
  error, truncated in the log, followed by a 45 s backoff) ->
  demo left partial, no video file. Sticky flash-tts; do NOT
  build with lite. Resume: chunks 2–4 ONLY, flash-tts, after
  quota refill.

### 37–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 35 chunks 2–4 (lite), 36 chunks 2–4 (flash),
  then 37–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Lite-tts: demo 35 chunk 2, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Flash-tts: demo 36 chunk 2, 3 attempts (45 s backoffs), the
  final attempt HTTP 429 RESOURCE_EXHAUSTED with the same
  description (one intermediate attempt failed with a non-quota
  transport error; see above). After demo 36 stopped, both
  models were marked exhausted and the run stopped cleanly.
  Builder exit 0.
- Calls this run (from `resume_results.json`): 2 generated
  (both flash-tts), 2 reused, 0 non-quota failures recorded as
  final outcomes. New audio tokens this run: 352 (36: 221 + 131;
  prompt 46).

## Drive sync (this run)

- None. No demo finished this run, so no folder listing check
  or upload was needed and none was made. Drive still holds
  the same 34 mp4s (01–34 contiguous) from earlier runs.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 31 (flash), 32 (lite),
  33 (lite), 34 (lite) — 34 of 42 demos.
- Partial: 35 (2/5, lite), 36 (2/5, flash). Untouched: 37–42
  (all model None, undecided). No 0-chunk sticky model record
  needed resetting this run. `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_1635.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run). No new frame checks (no demo built).
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 22 — cron resume run 2026-11-03 18:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (`find -mmin -20`
empty). Nothing was uploaded, moved, or deleted; no new video
file was produced. `output/demos/`, `output/demos_v4/`, and all
backup folders were not touched. Progress backup:
`demo_work_v5/progress.json.bak-1805` (taken before the run).

## Result per demo (this run)

### 35_EDATE — PARTIAL, 4 of 5 chunks (flash-lite-tts)

- Chunks 0–1 reused cached (7.09 s, 5.48 s). Chunks 2–3
  generated new this run, lite model, Achird, same style:
  chunk 2, 4.64 s (prompt 13, audio 116), text: "Result, bees
  June, do hazar pachees."; chunk 3, 10.57 s (prompt 36,
  audio 272), trim 0.31 s leading silence (0.05 s pad kept),
  text: "Agar date iktis January ho, to agla mahina athais
  February dega, kyunke iktis February hoti hi nahi. Ye theek
  hai, error nahi." Correct spelling EDATE kept. Chunk 4:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED -> demo left
  partial, no video file. Sticky flash-lite-tts; do NOT build
  with flash. Resume: chunk 4 ONLY, flash-lite-tts, after
  quota refill. Pending chunk 4 text: "Faida ye hai ke review
  aur renewal date aik formula mein mil jati hai."

### 36_DATEDIF — PARTIAL, 4 of 5 chunks (flash-tts)

- Chunks 0–1 reused cached (8.49 s, 5.24 s). Chunks 2–3
  generated new this run, flash-tts, Achird, same style:
  chunk 2, 2.64 s (prompt 7, audio 66), text: "Result, aath
  din."; chunk 3, 10.80 s (prompt 32, audio 270), text:
  "Agar start date end date se baad mein ho, to NUM error
  aata hai. Hal, order theek karein, pehle start, phir
  end." Correct spelling DATEDIF kept. Chunk 2 and chunk 3
  each succeeded on retry 2 after 45 s backoffs (rolling
  refill freed a call each time). Chunk 4: 3 attempts, all
  HTTP 429 RESOURCE_EXHAUSTED -> demo left partial, no video
  file. Sticky flash-tts; do NOT build with lite. Resume:
  chunk 4 ONLY, flash-tts, after quota refill. Pending
  chunk 4 text: "Faida ye hai ke task ki age exact dinon
  mein mil jati hai."

### 37–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 35 chunk 4 (lite), 36 chunk 4 (flash),
  then 37–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Lite-tts: demo 35 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Flash-tts: demo 36 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 36 chunks 2 and 3, retry 2 each). No
  retries were burned beyond the builder's 3-attempt rule; the
  run stopped as soon as both models reported exhausted.
  Builder exit 0.
- Calls this run (from `resume_results.json`): 4 generated
  (2 flash-tts + 2 flash-lite-tts), 4 reused, 0 non-quota
  failures. New audio tokens this run: 724 (35: 388, 36: 336;
  prompt 88).

## Drive sync (this run)

- None. No demo finished this run, so no folder listing check
  or upload was needed and none was made. Drive still holds
  the same 34 mp4s (01–34 contiguous) from earlier runs.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 31 (flash), 32 (lite),
  33 (lite), 34 (lite) — 34 of 42 demos.
- Partial: 35 (4/5, lite), 36 (4/5, flash). Untouched: 37–42
  (all model None, undecided). No 0-chunk sticky model record
  needed resetting this run. `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_1805.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run). No new frame checks (no demo built).
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 23 — cron resume run 2026-11-03 19:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (`find -mmin -20`
empty; last write 18:15 PKT, the Batch 22 build-report update).
Nothing was uploaded, moved, or deleted except the single Drive
upload of 35_EDATE.mp4 recorded below. `output/demos/`,
`output/demos_v4/`, and all backup folders were not touched.
Progress backup: `demo_work_v5/progress.json.bak-1935` (taken before
the run).

## Runtime restart during this run (no work lost, no chunk regenerated)

- The first builder invocation (19:35 PKT, log
  `demo_work_v5/cron_run_1935.log`) was killed by a service restart
  within about a minute: its log is empty (0 bytes, tee output lost
  with the kill), `resume_results.json` / `resume_log.txt` were never
  rewritten, and no chunk audio was generated. Its only effect was a
  progress.json rewrite recording the same pre-run state (34 complete,
  35 4/5 lite, 36 4/5 flash, 37–42 model None). No chunk was
  generated twice because of the kill: the second invocation reused
  the cache checks.
- The second invocation (`python3 -u build_v5_resume.py`, log
  `demo_work_v5/cron_run_1935b.log`) survived, finished cleanly on
  quota exhaustion at about 19:43 PKT (BUILDER_EXIT:0), and wrote
  `resume_log.txt` / `resume_results.json`. All results below are
  from the second invocation; the first invocation generated no
  deliverable audio that was counted. (Same pattern as Batch 6.)

## Result per demo (this run)

### 35_EDATE.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/35_EDATE.mp4`, 717,125 bytes, 33.71 s.
- Chunk durations (s): 7.09, 5.48, 4.64, 10.57, 5.68. Sum 33.46 s
  (difference 0.25 s, within 0.5 s). Chunks 0–3 reused cached;
  chunk 4 generated new this run, lite model, Achird, same style,
  after one 45 s-backoff retry on quota. Text: "Faida ye hai ke
  review aur renewal date aik formula mein mil jati hai." No trims.
  Correct spelling EDATE kept in all chunk texts.
- Tokens (all 5 chunks, raw JSON — the source of truth;
  `progress.json` `tokens` for this demo double-counts on cache
  reuse and is NOT used here): prompt 108, audio 851 (new chunk
  this run: prompt 17, audio 142).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -20.2 dB, max -1.8 dB. Focused frame:
  Column A WorkID / Column M DueDate - LOOKUP, IT-001 highlighted,
  20-May-2025 -> 20-Jun-2025. Result frame (v4 visual, kept
  verbatim): `=EDATE(DATE(2025,1,31),1)`, 31-Jan-2025 ->
  28-Feb-2025 with the red banner. Error frame: `=EDATE(
  '1_IT_PM_DATA'!M4,1)`, DueDate 20-May-2025 -> 20-Jun-2025 with
  the teal fix banner. Header "Demo 35 / 42", tabs fully visible,
  nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 36_DATEDIF — STILL PARTIAL, 4 of 5 chunks (flash-tts)

- Chunks 0–3 reused cached (8.49, 5.24, 2.64, 10.80 s). Chunk 4:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED -> demo left
  partial, no video file. Sticky flash-tts; do NOT build with
  lite. Resume: chunk 4 ONLY, flash-tts, after quota refill.
  Pending chunk 4 text (chunk text file present in
  `demo_work_v5/`): "Faida ye hai ke task ki age exact dinon
  mein mil jati hai." Correct spelling DATEDIF kept.

### 37_SWITCH — PARTIAL, 2 of 5 chunks (flash-lite-tts)

- Fresh demo, reached with flash-tts already exhausted, so the
  builder chose lite for the whole demo per the per-demo rule.
  Chunks generated new this run, lite model, Achird, same style:
  chunk 0, 6.00 s (prompt 21, audio 150), text: "Is demo mein
  hum SWITCH seekhenge. SWITCH aik value ko kayi cases se match
  karta hai."; chunk 1, 8.03 s (prompt 19, audio 207; trim
  0.25 s leading silence, 0.05 s pad kept), text: "Formula mein
  Status, phir Completed to one, In Progress to two, Delayed to
  four." The "IT-001" restoration is recorded in progress.json
  rewordings (Helio artifact removed). Chunk 2: 3 attempts, all
  HTTP 429 RESOURCE_EXHAUSTED (lite also exhausted) -> run
  stopped cleanly. No video file. The 2-chunk lite record is
  sticky and is NOT reset (unlike a 0-chunk record): resume
  chunks 2–4 ONLY, flash-lite-tts. Pending chunk 2 text (chunk
  text file present in `demo_work_v5/`): "Result, one, kyunke
  IT-001 Completed hai."

### 38–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 36 chunk 4 (flash), 37 chunks 2–4 (lite),
  then 38–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 36 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 37 chunk 2, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed once in this run: a single call freed
  up after a 45 s backoff (demo 35 chunk 4, retry 1). No retries
  were burned beyond the builder's 3-attempt rule; the run
  stopped as soon as both models reported exhausted.
- Calls this run (from `resume_results.json`, second invocation):
  3 generated (all flash-lite-tts), 8 reused, 0 non-quota
  failures. New audio tokens this run: 499 (35: 142, 37: 357;
  prompt 57).

## Drive sync (this run)

- Folder listing checked first (19:43 PKT, via
  `hatch_gws_cli drive files list --params`, saved as
  `demo_work_v5/drive_listing_1935.txt`): 34 mp4s present
  (01–34 contiguous), sizes matching the local files.
  35_EDATE.mp4 was NOT present, so only it was uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 35_EDATE.mp4`
  Upload returned Drive file ID
  `1T9mONmv-Nf87WIiXtEcz-6Aq79p91dZe`.
- Post-upload listing re-checked (19:43 PKT): 35_EDATE.mp4
  present with size 717,125 bytes, matching the local file
  exactly. No file was moved, deleted, or trashed in Drive;
  backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 31 (flash), 32 (lite),
  33 (lite), 34 (lite), 35 (lite) — 35 of 42 demos.
- Partial: 36 (4/5, flash), 37 (2/5, lite). Untouched: 38–42
  (all model None, undecided). No 0-chunk sticky model record
  needed resetting this run. `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_1935.log` (empty —
  killed with the service restart, see above),
  `demo_work_v5/cron_run_1935b.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_35/`, `demo_work_v5/drive_listing_1935.txt`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 24 — cron resume run 2026-11-03 21:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (`find -mmin -20`
empty; last write 19:44 PKT, the Batch 23 build-report update).
Nothing was uploaded, moved, or deleted except the two Drive uploads
of 36_DATEDIF.mp4 and 37_SWITCH.mp4 recorded below.
`output/demos/`, `output/demos_v4/`, and all backup folders were not
touched. Progress backup:
`demo_work_v5/progress.json.bak-2105` (taken before the run).

## Result per demo (this run)

### 36_DATEDIF.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/36_DATEDIF.mp4`, 678,137 bytes, 32.26 s.
- Chunk durations (s): 8.49, 5.24, 2.64, 10.80, 4.88. Sum 32.05 s
  (difference 0.21 s, within 0.5 s). Chunks 0–3 reused cached;
  chunk 4 generated new this run, flash-tts / Achird, same style.
  Text: "Faida ye hai ke task ki age exact dinon mein mil jati hai."
  No trims. Correct spelling DATEDIF kept in all chunk texts.
- Tokens (all 5 chunks, raw JSON — the source of truth):
  prompt 102, audio 810 (new chunk this run: prompt 17, audio 122).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.2 dB, max -1.5 dB. Focused frame:
  Column L StartDate / Column M DueDate - LOOKUP, IT-001
  12-May-2025 -> 20-May-2025, 8 days. The builder's "result"
  frame is mid chunk 3 (the v4 error case): wrong order,
  `=DATEDIF(...M4,...L4,"d")` -> #NUM! with the red banner. The
  builder's "error" frame is mid chunk 4: correct order,
  StartDate 12-May-2025, EndDate 20-May-2025 -> 8 days. Header
  "Demo 36 / 42", tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 37_SWITCH.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/37_SWITCH.mp4`, 626,594 bytes, 31.35 s.
- Chunk durations (s): 6.00, 8.03, 5.60, 6.88, 4.61. Sum 31.12 s
  (difference 0.23 s, within 0.5 s). Chunks 0–1 reused cached;
  chunks 2–4 generated new this run, lite model, Achird, same
  style. Chunk 2 needed two 45 s-backoff retries, chunk 3 one,
  chunk 4 two (rolling refill freed a call each time). Trim:
  chunk 4, 0.27 s leading silence (0.05 s pad kept). Chunk 2
  keeps the restored "IT-001" (Helio artifact removed, recorded
  in progress.json rewordings). Correct spelling SWITCH kept.
- Tokens (all 5 chunks, raw JSON): prompt 94, audio 791
  (new chunks this run: prompt 54, audio 434).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -20.0 dB, max -1.1 dB. Focused frame:
  Column A WorkID / Column G Status - LOOKUP, IT-001 Completed
  -> code 1. The builder's "result" frame is mid chunk 3 (the
  v4 error case): Status typed wrong, "complete" -> default 9
  with the red banner. The builder's "error" frame is mid
  chunk 4: Status Completed -> code 1. Header "Demo 37 / 42",
  tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 38_TEXTJOIN — PARTIAL, 4 of 5 chunks (flash-tts)

- Chunks generated new this run, flash-tts, Achird, same style:
  chunk 0, 7.37 s (prompt 27, audio 191; trim 0.27 s); chunk 1,
  7.48 s (prompt 16, audio 187), after one 45 s-backoff retry;
  chunk 2, 6.81 s (prompt 17, audio 177; trim 0.27 s); chunk 3,
  7.24 s (prompt 21, audio 181). Correct spelling TEXTJOIN kept
  in all chunk texts. Chunk 4: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED -> demo left partial, no video file.
  Sticky flash-tts; do NOT build with lite. Resume: chunk 4
  ONLY, flash-tts, after quota refill. Pending chunk 4 text
  (chunk text file present in `demo_work_v5/`): "Faida ye hai
  ke blockers ki aik line summary ban jati hai."

### 39_SUMPRODUCT — PARTIAL, 1 of 5 chunks (flash-lite-tts)

- Fresh demo, reached with flash-tts already exhausted, so the
  builder chose lite for the whole demo per the per-demo rule.
  Chunk 0 generated new this run, lite model: 6.16 s (prompt 19,
  audio 154), after one 45 s-backoff retry. Text: "Is demo mein
  hum SUMPRODUCT seekhenge. SUMPRODUCT weighted total nikalta
  hai." Chunk 1: 3 attempts, all HTTP 429 RESOURCE_EXHAUSTED
  (lite also exhausted) -> run stopped cleanly. No video file.
  The 1-chunk lite record is sticky and is NOT reset (unlike a
  0-chunk record): resume chunks 1–4 ONLY, flash-lite-tts.
  Pending chunk 1 text (chunk text file present in
  `demo_work_v5/`): "Formula mein Story Points, multiply,
  percent complete, phir divide, total Story Points."

### 40–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; all records still model None (undecided).
  Next run order: 38 chunk 4 (flash), 39 chunks 1–4 (lite),
  then 40–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 38 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 39 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed repeatedly in this run: single calls
  freed up after 45 s backoffs (demo 37 chunks 2, 3 and 4; demo
  38 chunk 1; demo 39 chunk 0). No retries were burned beyond
  the builder's 3-attempt rule; the run stopped as soon as both
  models reported exhausted. Builder exit 0.
- Calls this run (from `resume_results.json`): 9 generated
  (5 flash-tts + 4 flash-lite-tts), 6 reused, 0 non-quota
  failures. New audio tokens this run: 1,446 (36: 122, 37: 434,
  38: 736, 39: 154; prompt 171).

## Drive sync (this run)

- Folder listing checked first (via `hatch_gws_cli drive files
  list`, saved as `demo_work_v5/drive_listing_2105.json`): 35
  mp4s present (01–35 contiguous), sizes matching the local
  files. 36_DATEDIF.mp4 and 37_SWITCH.mp4 were NOT present, so
  only they were uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name <file>`.
  Uploads returned Drive file IDs: 36
  `1i4M8vExq-TNkLHsf-fNzdLMt3DB5mijo`, 37
  `1L3aFJuhnNWApIcjX0_WRpGu5IQxMkXrx`.
- Post-upload listing re-checked (saved as
  `demo_work_v5/drive_listing_2105_post.json`): 37 mp4s (01–37
  contiguous) with sizes matching the local files exactly (36:
  678,137 bytes; 37: 626,594 bytes). No file was moved, deleted,
  or trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 31 (flash), 32 (lite),
  33 (lite), 34 (lite), 35 (lite), 36 (flash), 37 (lite) —
  37 of 42 demos.
- Partial: 38 (4/5, flash), 39 (1/5, lite). Untouched: 40–42
  (all model None, undecided). No 0-chunk sticky model record
  needed resetting this run. `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_2105.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_36/`, `demo_work_v5/verify_37/`,
  `demo_work_v5/drive_listing_2105.json`,
  `demo_work_v5/drive_listing_2105_post.json`,
  `demo_work_v5/upload_36_2105.txt`,
  `demo_work_v5/upload_37_2105.txt`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 25 — cron resume run 2026-11-03 22:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (`find -mmin -20`
empty; last write 21:18 PKT, the Batch 24 build-report update).
Nothing was uploaded, moved, or deleted except the single Drive upload
of 38_TEXTJOIN.mp4 recorded below. `output/demos/`,
`output/demos_v4/`, and all backup folders were not touched. Progress backup:
`demo_work_v5/progress.json.bak-2235` (taken before the run).

## Result per demo (this run)

### 38_TEXTJOIN.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/38_TEXTJOIN.mp4`, 718,272 bytes, 33.50 s.
- Chunk durations (s): 7.37, 7.48, 6.81, 7.24, 4.36. Sum 33.26 s
  (difference 0.24 s, within 0.5 s). Chunks 0–3 reused cached;
  chunk 4 generated new this run, flash-tts / Achird, same style,
  after one 45 s-backoff retry on quota. Text: "Faida ye hai ke
  blockers ki aik line summary ban jati hai." Correct spelling
  TEXTJOIN kept in all chunk texts.
- Tokens (all 5 chunks, raw JSON — the source of truth):
  prompt 96, audio 846 (new chunk this run: prompt 15, audio 110).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -19.6 dB, max -1.1 dB. Focused frame:
  Text 1 / Joined columns, Login bug, (blank), API delay ->
  "Login bug, API delay". Result frame (v4 error case, mid
  chunk 3): `=TEXTJOIN(B2,FALSE,...)` -> "Login bug, , API delay"
  with the red banner. Error/fix frame: TRUE -> "Login bug, API
  delay". Header "Demo 38 / 42", tabs fully visible, nothing
  clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 39_SUMPRODUCT — STILL PARTIAL, 1 of 5 chunks (flash-lite-tts)

- Chunk 0 reused cached (6.16 s). Chunk 1: 3 attempts, all HTTP
  429 RESOURCE_EXHAUSTED (lite-tts) -> run continued to the next
  demo (the other model was still available at that point), no
  video file. Sticky flash-lite-tts; do NOT build with flash.
  Resume: chunks 1–4 ONLY, flash-lite-tts, after quota refill.
  Pending chunk 1 text: "Formula mein Story Points, multiply,
  percent complete, phir divide, total Story Points."

### 40_COUNTBLANK — PARTIAL, 1 of 5 chunks (flash-tts)

- Fresh demo, builder chose flash-tts first per the per-demo
  rule. Chunk 0 generated new this run, flash-tts: 5.04 s
  (prompt 21, audio 126). Text: "Is demo mein hum COUNTBLANK
  seekhenge. COUNTBLANK khaali cells ginta hai." Chunk 1:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (flash-tts) ->
  both models exhausted, run stopped cleanly. No video file.
  The 1-chunk flash record is sticky and is NOT reset (unlike a
  0-chunk record): resume chunks 1–4 ONLY, flash-tts.

### 41–42 — NOT ATTEMPTED in this run

- 0 chunks, no files; both records still model None (undecided).
  Next run order: 39 chunks 1–4 (lite), 40 chunks 1–4 (flash),
  then 41–42 in order (flash first per demo, lite only when
  flash is exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Lite-tts: demo 39 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Flash-tts: demo 40 chunk 1, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed once in this run: a single call freed
  up after a 45 s backoff (demo 38 chunk 4, retry 1). No retries
  were burned beyond the builder's 3-attempt rule; the run
  stopped as soon as both models reported exhausted. Builder
  exit 0.
- Calls this run (from `resume_results.json`): 2 generated
  (both flash-tts), 5 reused, 0 non-quota failures. New audio
  tokens this run: 236 (38: 110, 40: 126; prompt 36).

## Drive sync (this run)

- Folder listing checked first (via `hatch_gws_cli drive files
  list`, saved as `demo_work_v5/drive_listing_2235.json`):
  37 mp4s present (01–37 contiguous), sizes matching the local
  files. 38_TEXTJOIN.mp4 was NOT present, so only it was
  uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 38_TEXTJOIN.mp4`.
  Upload returned Drive file ID
  `12dccsy-zubdZ-50HwrSGGT3ej1JKx2l4`.
- Post-upload listing re-checked (saved as
  `demo_work_v5/drive_listing_2235_post.json`): 38 mp4s (01–38
  contiguous) with sizes matching the local files exactly (38:
  718,272 bytes). No file was moved, deleted, or trashed in
  Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 31 (flash), 32 (lite),
  33 (lite), 34 (lite), 35 (lite), 36 (flash), 37 (lite),
  38 (flash) — 38 of 42 demos.
- Partial: 39 (1/5, lite), 40 (1/5, flash). Untouched: 41–42
  (both model None, undecided). No 0-chunk sticky model record
  needed resetting this run. `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_2235.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_38/`, montage
  `demo_work_v5/verify_montage_38.png`,
  `demo_work_v5/drive_listing_2235.json`,
  `demo_work_v5/drive_listing_2235_post.json`,
  `demo_work_v5/upload_38_2235.txt`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio.


---

# Batch 26 — cron resume run 2026-11-04 00:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (`find -mmin -20`
empty; last write 22:42 PKT 3 Nov, the Batch 25 build-report update).
Nothing was uploaded, moved, or deleted except the single Drive upload
of 39_SUMPRODUCT.mp4 recorded below. `output/demos/`,
`output/demos_v4/`, and all backup folders were not touched. Progress backup:
`demo_work_v5/progress.json.bak-0005` (taken before the run).
Builder exit 0; run duration about 12.7 minutes.

## Result per demo (this run)

### 39_SUMPRODUCT.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/39_SUMPRODUCT.mp4`, 688,080 bytes, 32.35 s.
- Chunk durations (s): 6.16, 7.20, 6.14, 8.07, 4.56. Sum 32.13 s
  (difference 0.22 s, within 0.5 s). Chunk 0 reused cached;
  chunks 1–4 generated new this run, lite model, Achird, same
  style. Trims: chunk 2, 0.34 s; chunk 3, 0.29 s leading silence
  (0.05 s pad kept, recorded by the builder). Correct spelling
  SUMPRODUCT kept in all chunk texts. Chunks 1, 3 and 4 each
  needed 45 s-backoff retries (rolling refill freed a call).
- Tokens (all 5 chunks, raw JSON — the source of truth):
  prompt 102, audio 819 (new chunks this run: prompt 83,
  audio 665; chunk 0 cached, prompt 19 / audio 154, no new call).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) 0 gaps over
  2 s; volumedetect mean -19.7 dB, max -1.3 dB. Focused frame:
  Column A WorkID / Column O StoryPoints - LOOKUP / Column P
  %Complete - RESULT, IT-001..IT-005. Result/error frame (v4
  visual, mid chunk 3): mismatched ranges O4:O122 (119 rows)
  vs P4:P123 (120 rows) -> #VALUE! with the red banner.
  Fix frame: Story Points total 456, simple average 62.4%,
  Weighted %Complete 62.8%. Header "Demo 39 / 42", tabs fully
  visible, nothing clipped. PASS. Frames/montage in
  `demo_work_v5/verify_39/` and `verify_montage_39.png`.
- Drive: uploaded this run (see below); not previously in the folder.

### 40_COUNTBLANK — PARTIAL, 4 of 5 chunks (flash-tts)

- Chunks 0–3: 5.04 s (cached), 2.58 s, 7.68 s, 7.48 s
  (generated new this run, flash-tts; trim 0.27 s on chunk 1).
  Tokens for the 4 chunks (raw JSON): prompt 84, audio 576
  (new chunks this run: prompt 63, audio 450). Correct spelling
  COUNTBLANK kept. Chunk 4: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED -> demo left partial, no video file.
  Sticky flash-tts; do NOT build with lite. Resume: chunk 4
  ONLY, flash-tts, after quota refill. Pending chunk 4 text
  (chunk text file present in `demo_work_v5/`): "Faida ye hai
  ke data completeness aik number mein nazar aa jati hai."

### 41_IFNA — NOT BUILT, 0 chunks (model reset to undecided)

- Fresh demo, reached with flash-tts already exhausted, so the
  builder recorded it as lite before its first call; chunk 0:
  3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (lite also
  exhausted) -> run stopped cleanly. No chunk was generated.
  Its `progress.json` model was reset to None (undecided) after
  the run, with no chunk data lost, so the next run re-decides
  flash-first per the per-demo model order instead of treating
  the failed first call as a sticky lite choice. (Same treatment
  as demos 15/17 in earlier batches.) Chunk 1 text keeps the
  recorded "IT triple nine" -> "IT-999" restoration, coded in
  the resume builder.

### 42_SUBSTITUTE_CLEAN — NOT ATTEMPTED in this run

- 0 chunks, no files; record still model None (undecided).
  Next run order: 40 chunk 4 (flash), 41–42 in order (flash
  first per demo, lite only when flash is exhausted and the
  demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 40 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 41 chunk 0, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed repeatedly in this run: calls freed up
  after 45 s backoffs (demo 39 chunks 1, 3, 4; demo 40 chunks 1, 2).
  No retries were burned beyond the builder's 3-attempt rule; the
  run stopped as soon as both models reported exhausted.
- Calls this run (from `resume_results.json`): 7 generated
  (4 flash-lite-tts + 3 flash-tts), 2 reused, 0 non-quota
  failures. New audio tokens this run: 1,115 (39: 665, 40: 450;
  prompt 146).

## Drive sync (this run)

- Folder listing checked first (via `hatch_gws_cli drive files
  list`, saved as `demo_work_v5/drive_listing_0005.json`):
  39_SUMPRODUCT.mp4 was NOT present, so only it was uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 39_SUMPRODUCT.mp4`.
  Upload returned Drive file ID
  `1nFmQZ4q3gaUACiF2sRvZqga1sjGLtUmI`.
- Post-upload listing re-checked (saved as
  `demo_work_v5/drive_listing_0005_post.json`): the 39 entry is
  present with size 688,080 bytes, matching the local file
  exactly. No file was moved, deleted, or trashed in Drive;
  backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 31 (flash), 32 (lite),
  33 (lite), 34 (lite), 35 (lite), 36 (flash), 37 (lite),
  38 (flash), 39 (lite) — 39 of 42 demos.
- Partial: 40 (4/5, flash). Untouched: 41–42 (both model None,
  undecided; 41 reset this run, see above). `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_0005.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_39/`, montage
  `demo_work_v5/verify_montage_39.png`,
  `demo_work_v5/drive_listing_0005.json`,
  `demo_work_v5/drive_listing_0005_post.json`,
  `demo_work_v5/upload_39_0005.txt`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio. NEW FLAG for
  the next run: demo 42 chunk 1 text is "Formula mein IT dash
  zero zero one, spaces ke saath, ..." — that wording is NOT
  covered by the builder's restore list (only "IT zero zero one"
  in chunk 2 is), so check before generating demo 42 whether the
  spoken "IT dash zero zero one" wording is intended as-is; do
  not silently reword it.


---

# Batch 27 — cron resume run 2026-11-04 01:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (`find -mmin -20`
empty; last write 19:20 PKT 3 Nov, the Batch 26 build-report update).
Both locked TTS models re-listed before the run and still available
(`gemini-3.8-flash-tts`, `gemini-3.8-flash-lite-tts`). Nothing was
uploaded, moved, or deleted; no new video file was produced.
`output/demos/`, `output/demos_v4/`, and all backup folders were
not touched. Progress backup:
`demo_work_v5/progress.json.bak-0135` (taken before the run).
Builder exit 0; run duration about 6.4 minutes.

## Demo 42 wording check (Batch 26 flag — resolved before generating)

- Source texts read in `build_batch_D.py` (demo 42 SUBSTITUTE_CLEAN):
  chunk 1 "Formula mein IT dash zero zero one, spaces ke saath, phir
  dash ke aas paas ki space ko sirf dash se replace karein."; chunk 2
  "Result, IT zero zero one, saaf code."
- Determination: chunk 1 stays AS-IS. "IT dash zero zero one, spaces
  ke saath" describes the dirty pasted code "IT - 001" shown on
  screen (spaces around the dash); rewording it to "IT-001" would
  erase the dirty/clean distinction the demo teaches. It is not
  covered by the builder restore list and must not be silently
  reworded. Chunk 2 IS covered by the existing restore rule
  ("IT zero zero one" -> "IT-001"), so the clean result will be
  spoken as "IT-001" when generated — the same treatment as demos
  16/21. The demo 42 `progress.json` rewording entry for that rule
  pre-existed this run (present in `progress.json.bak-0135`, from
  the Batch 26 dry-check); demo 42 was not reached this run and no
  chunk text was changed.

## Result per demo (this run)

### 40_COUNTBLANK — STILL PARTIAL, 4 of 5 chunks (flash-tts)

- Chunks 0–3 reused cached (5.04, 2.58, 7.68, 7.48 s; gates
  re-passed at reuse). Chunk 4: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (flash-tts free-tier requests per day
  exceeded). No video file. Sticky flash-tts; do NOT build with
  lite. Resume: chunk 4 ONLY, flash-tts, after quota refill.
  Pending chunk 4 text (chunk text file present in
  `demo_work_v5/`): "Faida ye hai ke data completeness aik number
  mein nazar aa jati hai." No new API call was spent on this demo.

### 41_IFNA — PARTIAL, 3 of 5 chunks (flash-lite-tts)

- Fresh demo, reached with flash-tts already exhausted, so the
  builder chose lite for the whole demo per the per-demo rule
  (model recorded as lite before its first call; flash-first
  fallback inside `synth_for` was not available because flash
  was already in EXHAUSTED).
- Chunks generated new this run, lite model, Achird, same style:
  chunk 0, 8.39 s (prompt 31, audio 216; trim 0.25 s leading
  silence, 0.05 s pad kept; needed one 45 s-backoff retry on
  quota); chunk 1, 10.20 s (prompt 26, audio 255; needed two
  45 s-backoff retries); chunk 2, 2.60 s (prompt 6, audio 65;
  first attempt succeeded). Chunk 1 text keeps the restored
  "IT-999" (builder rewording recorded in progress.json:
  "'IT triple nine' -> 'IT-999'"). Chunk texts verified against
  the `chunk_V5D_41_*.txt` files in `demo_work_v5/`.
- Chunk 3: 3 attempts, all HTTP 429 RESOURCE_EXHAUSTED (lite
  also exhausted) -> run stopped cleanly. No video file. The
  3-chunk lite record is sticky and is NOT reset (unlike a
  0-chunk record): resume chunks 3–4 ONLY, flash-lite-tts.
  Pending chunk 3 text (chunk text file present):
  "Agar range ghalat ho, to wo error chhupa nahi, nazar aayega.
  Yehi is ka faida hai, asli ghalti chhupni nahi chahiye."

### 42_SUBSTITUTE_CLEAN — NOT ATTEMPTED in this run

- 0 chunks, no files; record still model None (undecided),
  rewording entry pre-existing (see wording check above). Next
  run order: 40 chunk 4 (flash), 41 chunks 3–4 (lite), then
  42 in order (flash first per demo, lite only when flash is
  exhausted and the demo has no flash chunks).

## Quota stop — exact evidence (both models exhausted)

- Flash-tts: demo 40 chunk 4, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: demo 41 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 41 chunk 0 retry 1, chunk 1 retry 2).
  No retries were burned beyond the builder's 3-attempt rule;
  the run stopped as soon as both models reported exhausted.
- Calls this run (from `resume_results.json`): 3 generated
  (all flash-lite-tts), 4 reused, 0 non-quota failures. New
  audio tokens this run: 536 (prompt 63), all demo 41 — sums of
  the per-chunk `chunk_V5D_41_*.raw.json` usageMetadata (the
  source of truth), matching the `progress.json` tokens field
  for demo 41 exactly.

## Drive sync (this run)

- None. No demo finished this run, so no folder listing check
  or upload was needed and none was made. Drive still holds
  the same 39 mp4s (01–39) from earlier runs; 40, 41, 42 are
  correctly absent locally and in Drive. No file was moved,
  deleted, or trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 31 (flash), 32 (lite),
  33 (lite), 34 (lite), 35 (lite), 36 (flash), 37 (lite),
  38 (flash), 39 (lite) — 39 of 42 demos.
- Partial: 40 (4/5, flash), 41 (3/5, lite). Untouched: 42
  (model None, undecided). No 0-chunk sticky model record
  needed resetting this run. `demo_work_v5/progress.json`
  records every finished chunk and is the source of truth; the
  resume builder regenerates nothing already present.
- Run artifacts: `demo_work_v5/cron_run_0135.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), `demo_work_v5/progress.json.bak-0135`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio. Demo 42
  wording flag is RESOLVED (see above): chunk 1 as-is, chunk 2
  restored to "IT-001" by the existing builder rule.


---

# Batch 28 — cron resume run 2026-11-04 03:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
Nov 3 19:11 PKT). Models re-listed before the run via the connector
helper: both `gemini-3.8-flash-tts` and `gemini-3.8-flash-lite-tts`
present. Nothing was uploaded, moved, or deleted except the single
Drive upload of 40_COUNTBLANK.mp4 recorded below.
`output/demos/`, `output/demos_v4/`, and all backup folders were
not touched. Progress backup:
`demo_work_v5/progress.json.bak-0305` (taken before the run).
Builder exit 0; run duration about 5.0 minutes.

## Result per demo (this run)

### 40_COUNTBLANK.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/40_COUNTBLANK.mp4`, 619,469 bytes, 28.15 s.
- Chunk durations (s): 5.04, 2.58, 7.68, 7.48, 5.16. Sum 27.94 s
  (difference 0.21 s, within 0.5 s). Chunks 0–3 reused cached;
  chunk 4 generated new this run, flash-tts / Achird, same style,
  after one 45 s-backoff retry on quota. Text: "Faida ye hai ke
  data completeness aik number mein nazar aa jati hai." No trims.
- Tokens (all 5 chunks, sums of the per-chunk `*.raw.json`
  usageMetadata, the source of truth; `progress.json` `tokens`
  for this demo double-counts on cache reuse and is NOT used
  here): prompt 100, audio 705. New chunk this run: prompt 16,
  audio 129.
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) no gap over
  2 s; volumedetect mean -20.1 dB, max -1.6 dB. Focused frame:
  Column A WorkID / Column G Status / Column N FinishDate -
  RESULT, (blank) rows highlighted, "baaki 72 khaali hain".
  Result frame: `=COUNTBLANK('1_IT_PM_DATA'!N4:N123)`, space-cell
  warning (segment 3 visual, kept verbatim). Error frame: 120
  rows, 48 Completed, Blank cells (result) 72, same COUNTBLANK
  formula, red/green banners as in the v4 rendering. Header
  "Demo 40 / 42", tabs fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 41_IFNA — STILL PARTIAL, 3 of 5 chunks (flash-lite-tts)

- Chunks 0–2 reused cached (8.39, 10.20, 2.60 s; gates
  re-passed at reuse). Chunk 3: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (lite-tts free-tier requests per day
  exceeded). No video file. Sticky flash-lite-tts; do NOT build
  with flash. Resume: chunks 3–4 ONLY, flash-lite-tts, after
  quota refill. Pending chunk 3 text (chunk text file present
  in `demo_work_v5/`): "Agar range ghalat ho, to wo error chhupa
  nahi, nazar aayega. Yehi is ka faida hai, asli ghalti chhupni
  nahi chahiye." Chunk 4: "Faida ye hai ke lookup safe hai, aur
  asli ghalti bhi nazar aati hai." No new API call was spent on
  this demo.

### 42_SUBSTITUTE_CLEAN — PARTIAL, 2 of 5 chunks (flash-tts)

- Chunks generated new this run, flash-tts / Achird, same style:
  chunk 0, 6.68 s (prompt 18, audio 167); chunk 1, 9.36 s
  (prompt 29, audio 234). Chunk 1 text kept AS-IS per the Batch
  27 resolved wording check: "Formula mein IT dash zero zero
  one, spaces ke saath, phir dash ke aas paas ki space ko sirf
  dash se replace karein." — the dirty/clean distinction is
  preserved. Chunk 2: 3 attempts, all HTTP 429
  RESOURCE_EXHAUSTED (flash-tts also exhausted) -> run stopped
  cleanly. No video file. The 2-chunk flash record is sticky
  and is NOT reset: resume chunks 2–4 ONLY, flash-tts. Chunk 2
  text file already carries the restored "IT-001" ("Result,
  IT-001, saaf code.") by the existing builder rule. New audio
  tokens this demo this run: 401 (prompt 47).

## Quota stop — exact evidence (both models exhausted)

- Lite-tts: demo 41 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Flash-tts: demo 42 chunk 2, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED with the same description.
- Rolling refill observed once in this run: a single call freed
  up after a 45 s backoff (demo 40 chunk 4, retry 1). No retries
  were burned beyond the builder's 3-attempt rule; the run
  stopped as soon as both models reported exhausted.
- Calls this run (from `resume_results.json`): 3 generated
  (all flash-tts: 40 chunk 4, 42 chunks 0–1), 7 reused, 0
  non-quota failures. New audio tokens this run: 530 (40: 129,
  42: 401; prompt 63).

## Drive sync (this run)

- Folder listing checked first (03:06 PKT, before the build,
  and again before uploading): 39 mp4s present (01–39),
  matching local sizes. 40_COUNTBLANK.mp4 was NOT present, so
  only it was uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 40_COUNTBLANK.mp4`
  Upload returned Drive file ID
  `1XvNQI0MQmJeO4GWihocEXilzWwWT92R5`.
- Post-upload listing re-checked: 40 mp4s (01–40) with sizes
  matching the local files exactly (40: 619,469 bytes). Demos
  41 and 42 are correctly absent. No file was moved, deleted,
  or trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 31 (flash), 32 (lite),
  33 (lite), 34 (lite), 35 (lite), 36 (flash), 37 (lite),
  38 (flash), 39 (lite), 40 (flash) — 40 of 42 demos.
- Partial: 41 (3/5, lite), 42 (2/5, flash). Untouched: none.
  No 0-chunk sticky model record needed resetting this run.
  `demo_work_v5/progress.json` records every finished chunk
  and is the source of truth; the resume builder regenerates
  nothing already present.
- Run artifacts: `demo_work_v5/cron_run_20261104_0305.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_40/`, Drive listings
  `demo_work_v5/drive_listing_0305_{pre,preupload,post}.json`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio. When 41 and
  42 are complete: build the combined video, upload it and the
  status note, verify the final folder listing, and remove this
  cron job per the task body.


---

# Batch 29 — cron resume run 2026-11-04 04:35 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
03:14 PKT, 81 minutes before the run). Models re-listed before the
run via the connector helper: both `gemini-3.8-flash-tts` and
`gemini-3.8-flash-lite-tts` present. Nothing was uploaded, moved,
or deleted except the single Drive upload of 41_IFNA.mp4 recorded
below. `output/demos/`, `output/demos_v4/`, and all backup folders
were not touched. Progress backup:
`demo_work_v5/progress.json.bak-0435` (taken before the run).
Builder exit 0; run duration about 4.1 minutes.

## Result per demo (this run)

### 41_IFNA.mp4 — PASS (completed this run, flash-lite-tts)

- File: `output/demos_v5/41_IFNA.mp4`, 735,455 bytes, 35.18 s
  (ffprobe 35.176 s).
- Chunk durations (s): 8.39, 10.20, 2.60, 8.32, 5.44. Sum 34.95 s
  (difference 0.23 s, within 0.5 s). Chunks 0–2 reused cached;
  chunks 3–4 generated new this run, lite model, Achird, same
  style. Chunk 4 needed one 45 s-backoff retry (transient quota
  429 on attempt 0, retry 1 succeeded — rolling refill). No trims.
- Tokens (all 5 chunks, sums of the per-chunk `*.raw.json`
  usageMetadata, the source of truth): prompt 122, audio 880.
  New chunks this run: prompt 59, audio 344 (chunk 3: 38/208;
  chunk 4: 21/136).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) found no
  silence segment at all, so no gap over 2 s; volumedetect mean
  -19.3 dB, max -1.1 dB. Focused frame: Column A WorkID /
  Column F Owner - LOOKUP, IT-001 -> Ayesha Khan, "IT-999 list
  mein nahi hai" note. Result frame: `=XLOOKUP(B2,...)`,
  IT-999 -> #N/A. Error frame: `=IFNA(XLOOKUP(...),"Not
  Found")`, IT-999 -> Not Found. Header "Demo 41 / 42", tabs
  fully visible, nothing clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 42_SUBSTITUTE_CLEAN — PARTIAL, 3 of 5 chunks (flash-tts)

- Chunks 0–1 reused cached (6.68 s, 9.36 s). Chunk 2 generated
  this run, flash-tts: 4.90 s (prompt 13, audio 129), after one
  45 s-backoff retry on quota; trim 0.26 s leading silence
  (0.05 s pad kept). Text keeps the restored "IT-001":
  "Result, IT-001, saaf code."
- Chunk 3: 3 attempts, all HTTP 429 RESOURCE_EXHAUSTED
  (flash-tts free-tier requests per day exceeded) -> demo left
  partial, no video file. Sticky flash-tts; do NOT build with
  lite. Resume: chunks 3–4 ONLY, flash-tts, after quota refill.
  Pending chunk 3 text (chunk text file present in
  `demo_work_v5/`): "Aik ehtiyaat, SUBSTITUTE har jagah replace
  karta hai, sirf pehli jagah nahi. Pehli jagah ke liye instance
  number dein." Chunk 4 text (from `build_batch_D.py` spec;
  no chunk text file yet): "Faida ye hai ke import kiye hue
  codes bulk mein theek ho jate hain."

## Quota stop — exact evidence (flash-tts exhausted; lite-tts NOT proven exhausted)

- Flash-tts: demo 42 chunk 3, 3 attempts (45 s backoffs), each
  HTTP 429 RESOURCE_EXHAUSTED: "Generate content requests per day
  limit exceeded on Free Tier."
- Lite-tts: NOT exhausted in this run. Its only 429 was a
  transient one on demo 41 chunk 4 attempt 0, recovered on
  retry 1. The builder stopped because demo 42 was the last
  demo and its sticky model (flash-tts) was exhausted; no
  fallback applies to a demo with cached flash chunks.
- Rolling refill observed again: single calls freed up after
  45 s backoffs (demo 41 chunk 4 retry 1, demo 42 chunk 2
  retry 1). No retries were burned beyond the builder's
  3-attempt rule.
- Calls this run (from `resume_results.json`): 3 generated
  (2 flash-lite-tts + 1 flash-tts), 5 reused, 0 non-quota
  failures. New audio tokens this run: 473 (41: 344, 42: 129;
  prompt 72).

## Drive sync (this run)

- Folder listing checked first (04:35 PKT, before the build):
  40 mp4s present (01–40), matching local sizes. A second
  listing check immediately before uploading (04:42 PKT)
  confirmed 41_IFNA.mp4 was NOT present, so only it was
  uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name 41_IFNA.mp4`
  Upload returned Drive file ID
  `1ma--kLhahsHG0rFEkWWZX4Ion9ydufM3`.
- Post-upload listing re-checked: 41 mp4s (01–41) with sizes
  matching the local files exactly (41: 735,455 bytes). Demo
  42 is correctly absent. No file was moved, deleted, or
  trashed in Drive; backup folders untouched.

## Resume state (for the next run / cron)

- Complete in Drive and locally: 01 (flash), 02 (flash), 03
  (flash), 04 (flash), 05 (lite), 06 (lite), 07 (lite), 08
  (lite), 09 (flash), 10 (flash), 11 (flash), 12 (lite),
  13 (lite), 14 (lite), 15 (flash), 16 (lite), 17 (lite),
  18 (flash), 19 (flash), 20 (lite), 21 (flash), 22 (flash),
  23 (flash), 24 (lite), 25 (flash), 26 (lite), 27 (lite),
  28 (lite), 29 (flash), 30 (flash), 31 (flash), 32 (lite),
  33 (lite), 34 (lite), 35 (lite), 36 (flash), 37 (lite),
  38 (flash), 39 (lite), 40 (flash), 41 (lite) — 41 of 42 demos.
- Partial: 42 (3/5, flash). Untouched: none. No 0-chunk
  sticky model record needed resetting this run.
  `demo_work_v5/progress.json` records every finished chunk
  and is the source of truth; the resume builder regenerates
  nothing already present.
- Run artifacts: `demo_work_v5/cron_run_20261104_0435.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_41/`, Drive listings
  `demo_work_v5/drive_listing_0435_{pre,preupload,post}.json`,
  `demo_work_v5/drive_upload_41.txt`,
  `demo_work_v5/progress.json.bak-0435`.
- Open items carried forward unchanged: Ali's verdict on demo 05
  (lite voice vs flash voice) still pending; demos 05–07
  header "Demo N / 7" + green error fill visual-only fix still
  pending as a final pass reusing existing audio. When 42 is
  complete: build the combined video, upload it and the status
  note, verify the final folder listing, and remove this cron
  job per the task body. Do NOT remove the cron job yet.


---

# Batch 30 — FINAL cron resume run 2026-11-04 06:05 PKT (this run)

Cron run `excel-v5-voice-rebuild`, builder `build_v5_resume.py`
unchanged (cache-aware, per-demo model, 4 s sleep, 3 attempts / 45 s
backoff). Overlap guard checked first: no file in
`output/demos_v5/` modified in the prior 20 minutes (last write
04:43 PKT, 82 minutes before the run). Models re-listed before the
run via the connector helper: both `gemini-3.8-flash-tts` and
`gemini-3.8-flash-lite-tts` present. Nothing was moved, deleted, or
trashed in Drive; backup folders untouched. Progress backup:
`demo_work_v5/progress.json.bak-0605` (taken before the run).
Builder exit 0. This run completed the rebuild: all 42 demos are now
built, verified, and in Drive, plus the combined video and status note.
The cron job is removed by this run per the task body.

## Result per demo (this run)

### 42_SUBSTITUTE_CLEAN.mp4 — PASS (completed this run, flash-tts)

- File: `output/demos_v5/42_SUBSTITUTE_CLEAN.mp4`, 746,909 bytes,
  35.42 s (ffprobe 35.424 s).
- Chunk durations (s): 6.68, 9.36, 4.90, 9.08, 5.16. Sum 35.18 s
  (difference 0.24 s, within 0.5 s). Chunks 0–2 reused cached;
  chunks 3–4 generated new this run, flash-tts / Achird, same
  style. Chunk 3 text: "Aik ehtiyaat, SUBSTITUTE har jagah replace
  karta hai, sirf pehli jagah nahi. Pehli jagah ke liye instance
  number dein." Chunk 4 text: "Faida ye hai ke import kiye hue
  codes bulk mein theek ho jate hain." No trims. Chunk 2 keeps
  the restored "IT-005"->"IT-001" rewording recorded in
  progress.json (Helio artifact removed; correct spelling restored).
- Tokens (all 5 chunks, sums of the per-chunk `*.raw.json`
  usageMetadata, the source of truth): prompt 221, audio 1817.
  New chunks this run: prompt 54, audio 356 (chunk 3: 34/227;
  chunk 4: 20/129).
- Verification (personally viewed this run): ffprobe h264
  1280x720 + AAC; silencedetect (noise=-40dB, d=2.0) reported no
  silence segment, so no gap over 2 s; volumedetect mean
  -19.9 dB, max -1.5 dB. Focused frame: Column A WorkID /
  Pasted "IT - 001" / Fixed "IT-001". Result frame:
  `=SUBSTITUTE(B2," - ","-",...)`, Pasted "IT - 001 - A" ->
  Fixed "IT-001-A". Error/fix frame: instance-number formula
  `=SUBSTITUTE(B2," - ","-",1)`, "IT-001 - A" with the fix
  banner. Header "Demo 42 / 42", tabs fully visible, nothing
  clipped. PASS.
- Drive: uploaded this run (see below); not previously in the folder.

### 01–41 — already complete, skipped by the builder

- No chunks regenerated. All 41 were already built, verified in
  earlier batches, and in Drive.

## Quota — no stop this run

- No 429 was hit. Calls this run (from `resume_results.json`):
  2 generated (both flash-tts, demo 42 chunks 3–4), 3 reused,
  0 failures. New audio tokens this run: 356 (prompt 54).

## Combined video (built this run)

- File: `output/demos_v5/00_All_42_Formulas_Combined.mp4`,
  30,795,884 bytes, 1504.77 s (25 min 5 s).
- Built with ffmpeg concat of the 42 demos_v5 files in order
  (01–42, absolute paths in `demo_work_v5/combined_v5_list.txt`),
  re-encoded: `-vf scale=1280:720,format=yuv420p -c:v libx264
  -r 30 -c:a aac`. Sum of the 42 individual durations: 1504.70 s;
  combined duration differs by 0.07 s. ffprobe: h264 1280x720
  + AAC. PASS.
- Drive: uploaded this run (see below).

## Status note (uploaded this run)

- `output/05_README_All_42_Gemini_Voice.txt` (1,680 bytes),
  written this run: v5 completion note with the per-demo model
  record, the verification summary, and the known demos 05–07
  header/fill visual note. Uploaded to Drive as
  `05_README_All_42_Gemini_Voice.txt`. A local copy stays in
  `output/` alongside the 00–04 README files.

## Drive sync (this run) — final verification

- Folder listing checked first (06:05 PKT, before uploading):
  52 items: 41 demos (01–41), 4 README notes, BUILD_REPORT.md,
  2 backup folders, 4 documents (2 PPTs, workbook, Word guide).
  42, the combined video, and the 05 note were NOT present, so
  only those three were uploaded, with:
  `/opt/hatch/bin/hatch_gws_cli drive +upload <absolute path>
  --parent 1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4 --name <file>`.
  Uploads returned Drive file IDs: 42
  `1n3NVBNDoqAis02CKuNC5JNGtOvM3pjAt`, combined
  `1TxWJ9owAk9pcKQGRSo6bVQUoj_kIC92h`, note
  `1e2tVMK3Bx2EOraT2OkqqApBKydbBaSUy`.
- Post-upload listing re-checked: 55 items — 43 videos (42 demos
  + combined) with sizes matching the local files exactly, 6
  text notes (00 + 02–05 READMEs), BUILD_REPORT.md, 2 backup
  folders, 4 documents. No file was moved, deleted, or trashed
  in Drive; backup folders untouched.

## Final per-demo model record (all 42, from progress.json)

- gemini-3.8-flash-tts (21): 01, 02, 03, 04, 09, 10, 11, 15, 18,
  19, 21, 22, 23, 25, 29, 30, 31, 36, 38, 40, 42.
- gemini-3.8-flash-lite-tts (21): 05, 06, 07, 08, 12, 13, 14, 16,
  17, 20, 24, 26, 27, 28, 32, 33, 34, 35, 37, 39, 41.
- Voice Achird and the exact style string on every chunk, one
  model per demo, never mixed inside a demo. Free tier only,
  no billing.

## Resume state — COMPLETE, nothing left

- Complete in Drive and locally: all 42 demos + combined video
  + status note. Partial: none. Untouched: none.
  `demo_work_v5/progress.json` records every chunk and is the
  source of truth.
- Run artifacts: `demo_work_v5/cron_run_20261104_0605.log`,
  `demo_work_v5/resume_log.txt`, `demo_work_v5/resume_results.json`
  (overwritten by this run), frame checks in
  `demo_work_v5/verify_42/`, Drive listings
  `demo_work_v5/drive_listing_0605_{pre,post}.json`,
  `demo_work_v5/drive_upload_{42,combined,note}.txt`,
  `demo_work_v5/progress.json.bak-0605`,
  `demo_work_v5/combined_v5_list.txt`.
- Open items carried forward unchanged (not part of this job's
  completion criteria): Ali's verdict on demo 05 (lite voice vs
  flash voice) still pending; demos 05–07 header "Demo N / 7" +
  green error fill visual-only fix still pending as a separate
  final pass reusing existing audio. The cron job
  `excel-v5-voice-rebuild` was removed by this run.
