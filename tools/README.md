# corpus tools

This directory carries the prebuilt `corpus-build` binary used by the
`corpus-refresh` GitHub Actions workflow to keep this repository in sync with
the Normattiva Open Data bulk API, plus the registry used as the verify gate.

## Usage

    ./corpus-build --changed-only --output . \
      --registry tools/registry/must_have.json \
      --verify --cache-dir /tmp/cache --commit-msg-file /tmp/commit-msg.txt

`corpus-build` has two modes:

- **full (default)** — rebuild every requested collection from scratch.
- **`--changed-only`** — incremental refresh. Collections whose Normattiva
  `dataCreazione` / act count are unchanged since the committed
  `manifest.json` are skipped (no download). Changed collections are
  downloaded and diffed against the committed per-collection `.entries.json`
  (zip entry -> sha256 of the raw entry); changed/new acts are converted and
  written, acts that vanished from the zip are deleted, and `manifest.json`
  is updated in place. A collection without a `.entries.json` is backfilled
  by hashing its zip without converting anything.

The build is deterministic: frontmatter `fetched_at` is the collection's
snapshot date, so re-converting unchanged acts produces byte-identical files
and the workflow never churns the repo.

Downloads are resumable (HTTP Range) and retry transient failures, so large
collections survive mid-stream connection resets.

`--zip-override DIR` (a directory of `<Collection>.zip` files) runs against
local zips instead of the API — used for drills.

`--commit-msg-file` writes an Italian commit message describing the changes,
for the workflow's `git commit -F`.

The registry at `tools/registry/must_have.json` is the verify gate: the
workflow refuses to push (and exits non-zero) when a must-have act is
missing or outside its article-count window.
