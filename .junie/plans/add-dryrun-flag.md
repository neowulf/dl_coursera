---
sessionId: session-261024-202750-15pv
---

# Technical Design

### Current Implementation
The entry point is `dl_coursera_run.py`. The `main()` function (line 225) parses CLI args, then calls:
1. `crawl()` — crawls the course/specialization and caches results.
2. `gather_dl_tasks()` — produces a list of download tasks from the crawled data.
3. `download(dl_tasks)` — actually downloads the files.

The `download()` function at line 175 is the single gate for all file downloads. Other flags like `--no-video` and `--quiz` follow the same pattern: argparse flag → passed through to the relevant function.

### Proposed Changes
**File: `dl_coursera_run.py`**

1. **Add `--dryrun` argument** (after `--no-video`, around line 276):
   ```python
   parser.add_argument(
       '--dryrun',
       action='store_true',
       help='crawl and list download tasks without actually downloading files',
   )
   ```

2. **Skip `download()` when dryrun is enabled** (around line 313):
   ```python
   if args['dryrun']:
       print(f'Dry run: {len(dl_tasks)} file(s) would be downloaded.')
   else:
       download(dl_tasks)
   ```

This follows the exact same pattern as the existing `--no-video` flag: a simple argparse boolean that gates a specific behavior in `main()`.


# Delivery Steps

### ✓ Step 1: Add --dryrun flag and skip download when enabled
Add the `--dryrun` CLI flag to `dl_coursera_run.py` and conditionally skip the `download()` call.

- Add `--dryrun` argument to the argparse parser in `main()` (after `--no-video`).
- Replace the unconditional `download(dl_tasks)` call with a conditional: when `args['dryrun']` is `True`, print a summary message (e.g. `Dry run: N file(s) would be downloaded.`) instead of calling `download()`.
- No other files need changes — the flag is entirely handled in `main()`.