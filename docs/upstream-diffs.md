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

## CI: build the test image on a hosted runner and skip CodSpeed

**Files:**
- `.github/workflows/_build-app.yml`
- `.github/workflows/shared-ci.yml`
- `Makefile`

**Reason:** `build-test-app` requested `ubuntu-large`, a larger runner that only
the upstream organization has. Without it the job queues indefinitely, and
every test job depending on it never starts. `shared-benchmark` uploads results
to CodSpeed, which requires a `CODSPEED_TOKEN` secret; without it the job fails
with `HTTP 401` on every pull request. The test image tag came from
`git rev-parse --short=7`, which returns a longer abbreviation when the seven
character prefix is ambiguous among the objects a clone contains: the build job
checks out shallowly and the test jobs in full, so the same commit produced two
tags and `docker compose` tried to pull an image that was never built.

Here the test image builds on `ubuntu-latest`, benchmarks run only in the
upstream repository, and the tag is cut explicitly to seven characters.

**Upstream status:** the runner, CodSpeed and tag changes are local to this
fork; the tag fix is a candidate to propose upstream
**Upstream PR:** none yet
**Owner:** @psturc

**Rebase notes:** Re-apply if upstream renames these jobs, introduces more
`ubuntu-large` runners, or changes how the image tag is derived.

## notifications: configurable GitHub app slug in the install-app warning

**Files:**
- `apps/worker/services/notification/notifiers/mixins/message/sections.py`

**Reason:** The comment notifier's "install the Codecov GitHub app" warning
hardcoded `https://github.com/apps/codecov/installations/select_target`. A
self-hosted deployment installs an app under its own slug, so the link pointed
at the wrong app. The slug is read from `github.app_slug` and defaults to
`codecov`; values that are not a valid slug fall back to the default, so a
configured value never reaches the Markdown link destination unvalidated.

**Upstream status:** Pending
**Upstream PR:** none yet
**Owner:** @psturc

**Rebase notes:** Only the warning branch of
`_write_install_github_app_warning` is modified. Tests asserting the default URL
stay valid because the default slug is `codecov`.

## Pending decision: Sentry webhook integration

The self-hosted deployment previously carried a Sentry (GlitchTip) webhook
integration: `apps/codecov-api/api/sentry/`,
`apps/codecov-api/codecov_auth/views/sentry.py`,
`apps/codecov-api/webhook_handlers/views/sentry.py` and
`libs/shared/shared/helpers/sentry.py`. Upstream has since restructured this
area and no longer ships those endpoints, so they are **not** carried here. Port
them only if the deployment actually registers the Codecov Sentry app;
otherwise they are dead code.
