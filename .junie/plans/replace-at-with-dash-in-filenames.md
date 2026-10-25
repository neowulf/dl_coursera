---
sessionId: session-261024-201437-1vij
---

# Requirements

### Overview
Replace the `@` character used as a separator in generated filenames with `-` (hyphen). This makes filenames more portable and avoids issues with tools that treat `@` specially.

### Scope
**In Scope:**
- Replace `@` with `-` in all filename format strings in `DLTaskGatherer.py`
- Update the documentation in `.junie/plans/download-english-subtitles.md` to reflect the new naming

**Out of Scope:**
- Any other uses of `@` in the codebase (e.g., Python decorators, GitHub Actions version tags like `actions/checkout@v4`)

### Expected File Naming After Change
- Video: `01 - .mp4` (was `01@.mp4`)
- Single subtitle: `01 - .srt` (was `01@.srt`)
- Multiple subtitles: `01 - .en.srt` (was `01@.en.srt`)
- HTML content: `01 - slug.html` (was `01@slug.html`)

# Delivery Steps

### ✓ Step 1: Replace @ with - in DLTaskGatherer.py filename format strings
Replace all 5 occurrences of `@` with ` - ` in filename format strings in `dl_coursera/DLTaskGatherer.py`:
- Line 83: `'%02d@%s'` → `'%02d - %s'`
- Line 116: `'%02d@%s.html'` → `'%02d - %s.html'`
- Line 177: `'%02d@.mp4'` → `'%02d - .mp4'`
- Line 188: `'%02d@.srt'` → `'%02d - .srt'`
- Line 190: `'%02d@.%s.srt'` → `'%02d - .%s.srt'`
- Line 261: `'%02d@%s.html'` → `'%02d - %s.html'`

### ✓ Step 2: Update documentation to reflect new naming convention
Update `.junie/plans/download-english-subtitles.md` to reflect the new file naming convention:
- Change `01@.srt` to `01 - .srt`
- Change `01@.en.srt` to `01 - .en.srt`
- Change `01@.zh-CN.srt` to `01 - .zh-CN.srt`