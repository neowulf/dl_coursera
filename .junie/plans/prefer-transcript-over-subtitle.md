---
sessionId: session-261024-202025-p83j
---

# Technical Design

### Current Implementation

- **`define.py:57`** — `URL_LECTURE_1` already requests `subtitlesTxt` from the Coursera API alongside `subtitles` and `subtitlesVtt`.
- **`Crawler.py:730-743`** — `crawl_lecture()` extracts only `subtitles` (timed SRT captions) from the API response. The `subtitlesTxt` (plain text transcripts) field is fetched but ignored.
- **`define.py:244-251`** — `Video` class stores `subtitles` as a dict `{lang: url}`.
- **`DLTaskGatherer.py:181-191`** — `_gather_video()` downloads subtitles as `.srt` files.
- **`DLTaskGatherer.py:207-231`** — `_select_subtitles()` selects which language(s) to download.

### Key Decisions

- Transcripts (`subtitlesTxt`, plain text `.txt`) are preferred over subtitles (`subtitles`, timed `.srt`) when both are available for a given language.
- If a transcript is unavailable for a language but a subtitle exists, the subtitle is downloaded as a fallback.
- The `--subtitles` CLI flag (language selection) continues to control which languages are downloaded, applying to both transcripts and subtitles.

### Proposed Changes

1. **`Crawler.py`** — In `crawl_lecture()`, also extract `subtitlesTxt` from the API response and store it in the `Video` object as `transcripts` (a dict `{lang: url}`).

2. **`define.py`** — Update `Video.__init__()` to accept an optional `transcripts` parameter.

3. **`DLTaskGatherer.py`** — In `_gather_video()`:
   - For each selected language, first check if a transcript (`.txt`) is available and download it.
   - If no transcript exists for that language, fall back to the subtitle (`.srt`).
   - Rename `_select_subtitles()` logic to handle both transcripts and subtitles, or add a parallel `_select_transcripts()` and combine results with fallback logic.

# Delivery Steps

### * Step 1: Extract transcripts from API in Crawler.py and update Video model
Add `subtitlesTxt` extraction in `crawl_lecture()` and store transcripts in the `Video` object.
- In `Crawler.py:crawl_lecture()`, after extracting `subtitles`, also iterate over `_('subtitlesTxt')` to build a `transcripts` dict `{lang: url}`.
- In `define.py:Video.__init__()`, add an optional `transcripts=None` parameter and store it as `self['transcripts']` when provided.
- Update the `Video()` constructor call in `crawl_lecture()` to pass the new `transcripts` dict.

### ✓ Step 2: Prefer transcript over subtitle in DLTaskGatherer._gather_video()
Update `_gather_video()` to download `.txt` transcripts when available, falling back to `.srt` subtitles.
- In `DLTaskGatherer.py:_gather_video()`, after calling `_select_subtitles()`, also check `video.get('transcripts')` for each selected language.
- For each selected language: if a transcript URL exists, download it as `.txt`; otherwise fall back to the subtitle URL and download as `.srt`.
- Update file naming: transcripts save as `%02d - .txt` (single language) or `%02d - .<lang>.txt` (multiple languages); subtitles retain `.srt` extension as fallback.