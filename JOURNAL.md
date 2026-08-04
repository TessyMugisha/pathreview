## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/106

**Issue title:** Shared test fixture for a sample user profile is missing from `tests/fixtures/`

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
Several integration tests are currently being skipped because they depend on
`tests/fixtures/sample_profiles/basic_profile.json`, a shared fixture file
that was deleted from the repo. The `tests/fixtures/` directory doesn't exist
at all right now, and `tests/conftest.py` only provides text fixtures
(`sample_resume_text`, `sample_readme_text`) — there's no shared user-profile
fixture. A successful fix restores the JSON file with a realistic sample
portfolio (a GitHub username, resume content, and two repos) matching the
app's profile schema, so the skipped tests run again and future tests have
consistent sample data to rely on.

**Branch name:** test/106-restore-basic-profile-fixture

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

**Issue checklist reasoning:**
Tier 1, scoped to a single file with no production code changes. The issue
description names the exact file to restore, existing conftest.py fixtures
show the pattern to follow, and success is verifiable by running the
previously-skipped tests. Maintainer estimate is 1–2 hours, which fits my
availability. Main risk is getting the JSON schema right, which I can check
against the profile model in the codebase.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:**
https://github.com/TessyMugisha/pathreview/commit/bff40cb8d271a916dffe146fb679aff7572782f6

**Reproduction summary:**
I ran `make test-unit` and confirmed the suite runs but no test references the
missing fixture — `grep -rn "basic_profile" tests/` and `grep -rn "sample_profiles"
tests/` both return no matches, `tests/fixtures/` does not exist, and
`tests/integration/` contains only `__init__.py`. `tests/conftest.py` provides only
`sample_resume_text` and `sample_readme_text`, confirming there is no shared
user-profile fixture. The 53 failures in the suite are pre-existing seeded bugs in
unrelated modules, not caused by this gap.

**PLAN.md:**
https://github.com/TessyMugisha/pathreview/blob/test/106-restore-basic-profile-fixture/PLAN.md

**Blockers or open questions:**
The issue describes skipped integration tests that don't exist in the current
codebase, so there's nothing to un-skip — I plan to add a consumer test alongside
the fixture so it isn't an orphaned file. I also need to confirm how the two repos
should be represented, since `Profile` has no repos field and repo data appears to
live in `IngestedSource` / `agent/tools/github_tool.py`.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
All three sub-tasks from PLAN.md's "Map" section are implemented on
`test/106-restore-basic-profile-fixture`:

1. `tests/fixtures/sample_profiles/basic_profile.json` — the fixture itself, a
   `profile` object mirroring the `Profile` model's columns plus a `repos` list of
   two entries (`name`, `description`, `language`, `url`, `readme_text`).
2. `tests/conftest.py` — new `sample_user_profile` fixture that loads the JSON via a
   `FIXTURES_DIR` path anchored on `__file__`, so it resolves no matter where pytest
   is invoked from.
3. `tests/unit/test_sample_profile_fixture.py` — the consumer test, which resolves
   the orphaned-file risk I flagged in Week 8.

I settled the Week 8 open question about repo shape: repos are not a `Profile`
column, so the fixture keeps them in a sibling `repos` list shaped to what
`agent/tools/github_tool.py` consumes (`name` usable as `repo_name`) and to what
`IngestedSource.raw_data` can hold as free-form JSON. No production code is touched.

**Next steps:**
- Re-run `make check` and `make test-unit` and confirm my three files add no new
  failures against the pre-existing baseline.
- Open the PR as a draft, fill in the template, and document the pre-existing
  failures explicitly.
- Ask for peer review in Slack, address feedback, then mark ready for review.

**Blockers:**
The suite has pre-existing failures from seeded bugs in unrelated modules (bias
detector, resume parser, review service, PII scrubber). None are in scope for #106,
so I'm recording the baseline count before and after my change and showing it
unchanged rather than trying to fix them.

---

### Check-in 2 (end of week)

**PR link:** [paste after opening the PR against ascherj/pathreview]

**Branch:** `test/106-restore-basic-profile-fixture`

**What you built:**
Restored the missing shared test fixture `basic_profile.json` with a realistic
sample portfolio, added a `sample_user_profile` pytest fixture in `tests/conftest.py`
that loads it, and added a unit test that asserts the fixture's shape against the
`Profile` model so the two can't drift apart.

**Tests added or updated:**
`tests/unit/test_sample_profile_fixture.py` (new) — nine tests covering that the
fixture loads, that its `profile` keys exactly match `Profile.__table__.columns`,
that ids parse as UUIDs, that string fields fit the model's column lengths, that
timestamps are timezone-aware, and that both repo entries carry the fields
downstream tooling reads. `tests/conftest.py` (updated) — added the
`sample_user_profile` fixture.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes

Recorded against the documented pre-existing baseline, per the module guidance that
"passes" means my changes introduce no new failures. Unit tests: 53 failed / 375
passed before, 53 failed / 384 passed after — same 53 failures, plus my 9 new passing
tests. `ruff check .`: 182 errors before and after. `black --check .`: 52 files before
and after. My three files are clean under both `ruff check` and `black --check`.

Note on tooling: `make` isn't available in PowerShell on my machine, so I ran the
underlying commands directly (`.venv\Scripts\pytest tests/unit -v -m unit`,
`.venv\Scripts\ruff check .`). I deliberately did not run `make check` as a whole,
because its `format` target is `black .` rather than `black --check .` and would have
rewritten 52 unrelated files into my diff.

**Draft PR feedback received from:** [name or Slack handle, or "none"]