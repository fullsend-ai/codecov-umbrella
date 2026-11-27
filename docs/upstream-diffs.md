# Upstream divergence tracking

This file tracks modifications to files originating from
[codecov/umbrella](https://github.com/codecov/umbrella) that must be maintained
during upstream syncs.

Additions that upstream has no use for are not tracked here. The UBI based
`Containerfile.api` and `Containerfile.worker`, `.tekton/` and `deploy/` live
only on the `internal` branch of the GitLab mirror, never on `main`, so they
never take part in an upstream merge.

## reports: use the serialized chunk index in `ReadOnlyReport.from_chunks`

**Files:**
- `libs/shared/shared/reports/readonly.py`

**Reason:** `from_chunks` built the rust parser's filename → chunk mapping from
the insertion order of the files dict (`enumerate(inner_report._files.keys())`).
The index that the rust parser needs is the one recorded in each file's summary
(`files[name][0]`), which serialization produces explicitly and which does not
always match insertion order. When the two disagreed, lines were read from the
wrong chunk.

**Upstream status:** Pending
**Upstream PR:** none yet
**Owner:** @psturc

**Rebase notes:** Watch for changes to `from_chunks`, to the shape of the
`files` argument, or to `shared/reports/serde.py`, which builds
`{name: [index, totals, None, diff_totals]}` for each file, or
`{name: [index, None]}` when totals are omitted. The chunk index is element 0 in
both shapes.
