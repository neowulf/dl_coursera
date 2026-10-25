---
sessionId: session-261024-201127-19ho
---

# Current State & Usage

## Current State

The functionality to download English subtitles **already exists** in `dl_coursera`. Here's how it works:

### Default Behavior
When you run `dl_coursera` without the `--subtitles` flag, the tool automatically downloads English subtitles if they are available for a course. This is handled in `DLTaskGatherer._select_subtitles()` (lines 207-216):
- If only one subtitle language is available → download it
- If multiple languages are available and English (`en`) is one of them → download English
- Otherwise → no subtitles downloaded

### Explicit English Subtitles
You can also explicitly request English subtitles:
```
dl_coursera --cookies cookies.txt --outdir output --subtitles en <slug>
```

### How It Works (Architecture)
1. **Crawler** (`Crawler.py`, lines 728-743): Fetches subtitle URLs from Coursera's API for each video. The API endpoint `URL_LECTURE_1` requests `subtitles` fields, which return a dict of `{language_code: url}`.
2. **DLTaskGatherer** (`DLTaskGatherer.py`, lines 181-191): For each video, calls `_select_subtitles()` to determine which languages to download, then creates download tasks (`.srt` files).
3. **Downloader** (`dl_coursera_run.py`, line 311): Passes the subtitle language preference to `gather_dl_tasks()`.

### File Naming
- Single subtitle language: `01 - .srt` (no language suffix)
- Multiple languages: `01 - .en.srt`, `01 - .zh-CN.srt` (language suffix included)

## Scope

**No code changes are needed.** The feature is already implemented and working. If you're experiencing issues with English subtitles not being downloaded, it could be because:
- The course doesn't offer English subtitles
- The cookies have expired
- There's a network issue

If you'd like to modify the behavior (e.g., always download all available subtitles, or change the default language), please let me know the specific requirement.