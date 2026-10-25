---
sessionId: session-261024-203032-rgwa
---

# Requirements

### Overview
Change the default behavior of the `dl_coursera` CLI so that it runs in a safe, non-downloading mode by default.

### Proposed Changes
Three command-line flag defaults in `dl_coursera_run.py` need to be updated:

1. **`--dryrun`** — change default from `False` to `True`. Add a `--no-dryrun` flag to disable dry run.
2. **`--no-video`** — change default from `False` to `True`. Add a `--video` flag to re-enable video downloads.
3. **`--download`** — add a new flag defaulting to `False`. When `True`, actually download files (overrides dry run behavior).

This means the tool will, by default, only crawl and list what *would* be downloaded without actually downloading anything or downloading videos.

# Technical Design

### Current Implementation
In `dl_coursera_run.py`, the `main()` function defines CLI arguments using `argparse`:

- `--dryrun` (line 277): `action='store_true'` — defaults to `False`
- `--no-video` (line 272): `action='store_true'` — defaults to `False`
- No `--download` flag exists; downloading happens when `dryrun` is `False` (line 318-321)

The logic at lines 318-321 checks `args['dryrun']` to decide whether to download:
```python
if args['dryrun']:
    print(f'Dry run: {len(dl_tasks)} file(s) would be downloaded.')
else:
    download(dl_tasks)
```

### Proposed Changes
In `dl_coursera_run.py`:

1. **`--dryrun`**: Change from `action='store_true'` to `action='store_false'` with a complementary `--no-dryrun` flag. This makes dry run the default (`True`).
2. **`--no-video`**: Change from `action='store_true'` to `action='store_false'` with a complementary `--video` flag. This makes `no_video` default to `True`.
3. **`--download`**: Add new flag with `action='store_true'` defaulting to `False`. When `True`, it overrides dry run — files are actually downloaded.

Update the download decision logic (lines 318-321) to check both `dryrun` and `download`:
- If `download` is `True`, download files
- If `dryrun` is `True` (and `download` is `False`), print dry run message
- Otherwise, download files

### Files Modified
- `dl_coursera_run.py` — argument parser definitions and download logic

# Delivery Steps

### ✓ Step 1: Update CLI flag defaults in dl_coursera_run.py
Change the three CLI flag defaults and update the download decision logic.

- Change `--dryrun` from `action='store_true'` to `action='store_false'` with added `--no-dryrun` complementary flag (default dryrun=True)
- Change `--no-video` from `action='store_true'` to `action='store_false'` with added `--video` complementary flag (default no_video=True)
- Add new `--download` flag with `action='store_true'` (default False)
- Update the download decision logic at lines 318-321 to check `args['download']` in addition to `args['dryrun']`: download when `download` is True, otherwise show dry run message