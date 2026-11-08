# Excel for IT Project Management — Training Material

Training material by **Muhammad Bilal** — Excel formulas explained with real IT project-management data, for beginners. Narration in Roman Urdu; formula names, column names and on-screen text in English.

## What is in this repo

| File | What it is |
|---|---|
| `Operations_Analyst_IT_Project_Excel_Workbook.xlsx` | Rebuilt workbook. Main dataset: IT project data sheet `1_IT_PM_DATA`, rows 4–123 (120 work items). Original Jan–Jun 2025 dataset kept as appendix. |
| `Operations_Analyst_IT_Excel_Detailed_PPT.pptx` | Detailed presentation, 48 slides — one section per formula. |
| `Operations_Analyst_IT_Excel_Concise_PPT.pptx` | Concise presentation, 45 slides — one section per formula. |
| `Operations_Analyst_IT_Project_Learning_Guide.docx` | Word learning guide. |
| `BUILD_REPORT.md` | Per-demo verification for the v5 Gemini Achird rebuild: durations, results, flags and chunk records. |
| `demos/` (optional) | The 42 individual demo videos, `01_XLOOKUP.mp4` … `42_SUBSTITUTE_CLEAN.mp4` (29.3 MB total, no file over 1.1 MB). See “Videos” below. |

## Watch the demos

- **YouTube playlist (Unlisted):** [ADD PLAYLIST LINK AFTER UPLOAD]
- **Google Drive folder (Viewer-only):** https://drive.google.com/drive/folders/1OpKPzwcD3-lO02OSnOWxCktGsZwuGY-4
- **Combined video:** all 42 demos in order, 25 min 05 s, in the Drive folder (`00_All_42_Formulas_Combined.mp4`).

## The 42 demos

All demos use one voice, one language and one design: Google Gemini TTS voice Achird (warm, friendly, patient-teacher style), Roman Urdu narration, simpler 2-view design (focused columns with real Excel column letters, then the working sheet). No full 16-column sheet view. 21 demos used gemini-3.8-flash-tts and 21 used gemini-3.8-flash-lite-tts — same voice and style on both, never mixed inside one demo. Results below are the verified results from `BUILD_REPORT.md`.

| # | Formula | Result in the demo |
|---|---|---|
| 01 | XLOOKUP | Ayesha Khan; IT-999 → Not Found |
| 02 | VLOOKUP | Customer Portal Revamp; IT-999 → #N/A |
| 03 | INDEX-MATCH | 20 May 2025; IT-999 → #N/A |
| 04 | SUMIFS | 96.5 hours; wrong project → 0 |
| 05 | MATCH | Position 5; IT-999 → #N/A |
| 06 | SUMIF | 312.3 hours; wrong project → 0 |
| 07 | COUNTIF | 18; wrong status → 0 |
| 08 | COUNTIFS | 5 (Delayed + Critical); mismatched ranges → #VALUE! |
| 09 | AVERAGEIF | 13.025 hours (Completed); wrong status → #DIV/0! |
| 10 | AVERAGEIFS | 3.7 hours (average Variance, Customer Portal + Delayed); no match → #DIV/0! |
| 11 | IF | Variance 5.0 → Overdue; reversed sign → On Track (wrong label) |
| 12 | IFS | Delayed + 5.0 → Red; no TRUE fallback → #N/A |
| 13 | AND / OR | Delayed + Critical → ESCALATE; “Delayd” → Monitor |
| 14 | IFERROR | Not Found |
| 15 | TEXT | May-2025 |
| 16 | CONCAT / & | IT-001 - Customer Portal Revamp |
| 17 | LEFT / MID / RIGHT | IT \| 001 |
| 18 | LEN / TRIM / UPPER | Ayesha Khan (cleaned) |
| 19 | DATE / MONTH / YEAR | 5 (month) |
| 20 | EOMONTH | 30-Jun-2025 |
| 21 | NETWORKDAYS | 7 working days (12–20 May) |
| 22 | UNIQUE | 4 projects, spilled list |
| 23 | SORT / SORTBY | IT-110, 7.5 first (descending) |
| 24 | FILTER | 5 rows; no match → #CALC! |
| 25 | MAX / MIN / LARGE / SMALL | MAX 7.5, MIN −19.5, LARGE(3) 5.9, SMALL(1) −19.5; k=200 → #NUM! |
| 26 | RANK | HRMS 1, Customer 2, Service Desk 3, Inventory 4 |
| 27 | PERCENTILE | 20.51 hours; k=95 → #NUM! |
| 28 | References (relative / absolute / mixed) | Relative copied → 0; absolute/mixed → 26,700 PKR |
| 29 | Pivot via SUMIFS / COUNTIFS | COUNTIFS matrix, 30/project, 20/sprint, 120 total |
| 30 | Data Validation | “Completed” accepted; “done” rejected |
| 31 | Conditional Formatting | Delayed red, Completed green, Variance 3-colour scale |
| 32 | Power Query | Steps/flow demo only — see honesty note below |
| 33 | TODAY | 51 days overdue (report date fixed 30-Jun-2025) |
| 34 | WORKDAY | 21-May-2025 |
| 35 | EDATE | 20-Jun-2025 |
| 36 | DATEDIF | 8 days |
| 37 | SWITCH | Code 1; wrong spelling → default 9 |
| 38 | TEXTJOIN | “Login bug, API delay” |
| 39 | SUMPRODUCT | 62.8% weighted %Complete |
| 40 | COUNTBLANK | 72 (FinishDate blanks: 120 rows − 48 Completed) |
| 41 | IFNA | Not Found |
| 42 | SUBSTITUTE / CLEAN | IT001-A (from IT-001-A) |

Dataset totals (workbook): 120 items — Completed 48 (40%), Delayed 18 (15%); Planned 1,404.8 h, Actual 1,217.8 h.

## Videos

GitHub is the professional record, not the video home: no playlist and poor playback here. Watch on YouTube (playlist above) or in the Drive folder.

- Including the 42 individual videos adds 29.3 MB; the combined 25:05 video is a further 30.8 MB. Do **not** add the combined video to this repo — it is already in Drive and on YouTube. With docs only, this repo is under 1 MB.
- All videos are h264 1280×720 with audio, checked: no silence gap over 2 seconds in any file.

## Honesty notes — read before reusing a result

- The demos are Excel-screen videos built from the actual workbook. They are **not** literal operating-system screen recordings (LibreOffice screen capture is broken in this VM).
- Demo 32 (Power Query) is a steps-and-flow demo only. No live monthly files exist, so no live import is shown or faked; this is stated on screen and in the narration.
- Some results were verified by applying the formula in Python to workbook-verified inputs, or are literal-string / rule examples, rather than run through live Excel. Each case is flagged per demo in `BUILD_REPORT.md`.
- Two corrections are shown honestly in the demos: COUNTBLANK is 72, not 0 (72 FinishDate cells are blank), and the CONCAT serial shown is 45797.
- Voice: Google Gemini TTS, voice Achird. Correct Excel spellings were used in the voice input (SUMIFS, COUNTIFS, IT-005 and so on — no spaced-out pronunciation spellings). Gemini TTS is token-billed, not free; this rebuild ran on the free tier only, with no billing. Note: in this voice EOMONTH is hard to hear — the on-screen formula carries it.
- Previous demos in Ali’s own voice are kept as backup only (Drive folder `Backup_v3_Ali_Voice_Do_Not_Use`); they are not in this repo.

## How to use this material

For self-study, in order 01 → 42. Each demo shows: what the formula says, how to apply it to the dataset, the result, the common error and its fix, and the benefit. The concise PPT is the quick reference; the detailed PPT and Word guide explain each formula for a beginner.
