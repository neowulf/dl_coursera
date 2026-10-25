---
sessionId: session-261024-200604-1doo
---

# Requirements

### Overview
Add a `--no-video` command-line flag that tells the script to skip downloading video files while still downloading all other content (subtitles, reading materials, quizzes, slides, etc.).

### Scope

**In Scope:**
- Add `--no-video` CLI argument to `dl_coursera_run.py`
- Skip video (`.mp4`) download tasks in `DLTaskGatherer._gather_video()` when the flag is set
- Skip playlist (`.m3u`) generation when videos are skipped (since the playlist only lists videos)
- Keep subtitle downloads working independently (subtitles are still useful without videos)

**Out of Scope:**
- Skipping other media types (slides, reading materials, quizzes remain unchanged)
- Changing the Crawler behavior (crawling still collects all metadata)

# Technical Design

### Current Implementation

The download pipeline flows through three stages:
1. **`crawl()`** in `dl_coursera_run.py` — crawls Coursera and returns course/specialization metadata (includes video URLs)
2. **`gather_dl_tasks()`** in `dl_coursera_run.py` — calls `DLTaskGatherer.gather()` which builds a list of download tasks. The `_gather_video()` method (line 174) adds video `.mp4` download tasks and playlist entries, plus subtitle `.srt` tasks
3. **`download()`** in `dl_coursera_run.py` — executes the download tasks

### Proposed Changes

**`dl_coursera_run.py`:**
- Add `--no-video` argument (boolean flag, `action='store_true'`) to the argparse parser in `main()`
- Pass `no_video` to `gather_dl_tasks()`
- Update `gather_dl_tasks()` signature to accept `no_video` and pass it to `DLTaskGatherer`

**`DLTaskGatherer.py`:**
- Update `__init__` to accept `no_video=False` parameter
- In `_gather_video()`: when `no_video` is True, skip adding the video download task (`_add_dl_task` for `.mp4`) and skip adding to `_playlist_entries`, but still process subtitles
- In `_gather_playlist()`: skip generating the playlist when `no_video` is True (no videos = empty playlist)

# Delivery Steps

### ✓ Step 1: Add --no-video CLI flag and wire it through to DLTaskGatherer
Add the `--no-video` argument to the CLI parser and thread it through the call chain to `DLTaskGatherer`.

### ✓ Step 2: Skip video downloads and playlist in DLTaskGatherer when --no-video is set
Update `_gather_video()` to skip video `.mp4` download tasks and playlist entries when `no_video` is True, while keeping subtitle downloads. Skip playlist generation in `_gather_playlist()` when no videos are present.