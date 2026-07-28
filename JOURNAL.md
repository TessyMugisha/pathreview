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

**Reproduction commit link:** [paste after you commit]

**Reproduction summary:**
I ran `make test-unit` and confirmed the suite runs but no test references the
missing fixture — `grep -rn "basic_profile" tests/` and `grep -rn "sample_profiles"
tests/` both return no matches, `tests/fixtures/` does not exist, and
`tests/integration/` contains only `__init__.py`. `tests/conftest.py` provides only
`sample_resume_text` and `sample_readme_text`, confirming there is no shared
user-profile fixture. The 53 failures in the suite are pre-existing seeded bugs in
unrelated modules, not caused by this gap.


**Blockers or open questions:**
The issue describes skipped integration tests that don't exist in the current
codebase, so there's nothing to un-skip — I plan to add a consumer test alongside
the fixture so it isn't an orphaned file. I also need to confirm how the two repos
should be represented, since `Profile` has no repos field and repo data appears to
live in `IngestedSource` / `agent/tools/github_tool.py`.