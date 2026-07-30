# Cycle 3 — Rafter CLI Issue #35

## Phase I: Issue Selection

### Status

Phase I Complete — selected a new Cycle 3 issue after pivoting away from ExternalDNS #5151.

### Selected Issue

**Project:** Rafter CLI  
**Repository:** https://github.com/RafterSecurity/rafter-cli  
**Issue:** https://github.com/RafterSecurity/rafter-cli/issues/35  
**Issue Title:** ci: add stale issue/PR cleanup workflow  
**Fork:** https://github.com/kietcoderlor/rafter-cli  
**Working Branch:** https://github.com/kietcoderlor/rafter-cli/tree/fix-35-stale-workflow  

### Why I Chose This Issue

I selected this issue as my Cycle 3 issue because it has a small and clear scope. The issue asks for a GitHub Actions workflow that automatically marks inactive issues and pull requests as stale and eventually closes stale issues.

This issue is a better fit for the remaining weeks than my previous ExternalDNS issue because it has a concrete deliverable, a specified file to create, and clear configuration requirements. It is labeled as a good first issue, has no assignee, and had no linked branch or pull request when I selected it.

### Issue Claim

I commented on the GitHub issue to express interest in working on it for CodePath AI301.

I also marked/commented on the course Google Sheet row so other students know I am working on this issue.

---

## Phase II: Reproduce & Plan

### Status

Phase II Complete — local setup completed, issue verified, and solution plan drafted.

### Environment Setup

I set up the repository locally using Windows and Antigravity.

**Environment:**

- OS: Windows
- Editor: Antigravity
- Local path: `D:\codepath\rafter-cli`
- Fork: https://github.com/kietcoderlor/rafter-cli
- Branch: `fix-35-stale-workflow`
- Upstream: https://github.com/RafterSecurity/rafter-cli

**Setup commands used:**

```bash
cd D:\codepath
git clone https://github.com/kietcoderlor/rafter-cli.git
cd rafter-cli
git remote add upstream https://github.com/RafterSecurity/rafter-cli.git
git checkout -b fix-35-stale-workflow
```

### Reproduction / Verification Process

This issue is not a runtime bug. It is a missing repository automation workflow.

To verify the current state, I inspected the `.github/workflows` directory and checked whether the requested stale workflow already exists.

Commands used:

```powershell
cd D:\codepath\rafter-cli
dir .github
dir .github\workflows
Test-Path .github\workflows\stale.yml
```

### Current Behavior

The repository does not currently have the requested workflow file:

```text
.github/workflows/stale.yml
```

Existing workflow files found in `.github/workflows`:

```text
publish.yaml
test-action.yml
test-comprehensive.yml
test-github-action.yml
validate-release.yml
```

Because the stale workflow is missing, inactive issues and pull requests are not automatically labeled as stale according to the policy described in issue #35.

### Expected Behavior

The repository should have a GitHub Actions workflow at:

```text
.github/workflows/stale.yml
```

The workflow should use:

```text
actions/stale@v9
```

Expected behavior from issue #35:

- Issues with no activity for 60 days get labeled `stale`.
- Stale issues with no activity for 14 more days get closed.
- Pull requests with no activity for 30 days get labeled `stale`.
- Issues or PRs labeled `pinned` or `security` are exempt.
- The `stale` label is removed when there is new activity.

### Files to Modify

Planned file to create:

```text
.github/workflows/stale.yml
```

### Repository Conventions Observed

The existing workflows use clear descriptive names and standard GitHub Actions structure.

Patterns observed:

- Workflow names are descriptive, such as `Test Composite Action`.
- Workflows use explicit triggers such as `push`, `pull_request`, and `workflow_dispatch`.
- Jobs run on `ubuntu-latest`.
- Existing workflow files mostly use the `.yml` extension, and the issue specifically requests `stale.yml`.

### Solution Plan

1. Create `.github/workflows/stale.yml`.
2. Configure it to run on a daily `schedule`.
3. Add `workflow_dispatch` so maintainers can run it manually.
4. Use `actions/stale@v9`.
5. Set issue stale timing:
   - `days-before-issue-stale: 60`
   - `days-before-issue-close: 14`
6. Set PR stale timing:
   - `days-before-pr-stale: 30`
7. Exempt items labeled:
   - `pinned`
   - `security`
8. Configure the workflow so the `stale` label is removed when new activity occurs.
9. Verify YAML syntax and keep the PR focused only on the workflow file.

### Implementation Note for Phase III

The issue clearly requests stale labeling for PRs after 30 days, but it does not explicitly say that stale PRs should also be closed after 14 more days. For this reason, I avoided adding PR auto-close behavior during implementation.

### Phase II Checklist

- [x] Repository forked.
- [x] Local repository cloned.
- [x] Working branch created.
- [x] Project opened in Antigravity.
- [x] Existing `.github/workflows` directory inspected.
- [x] Missing `stale.yml` workflow verified.
- [x] Current behavior documented.
- [x] Expected behavior documented.
- [x] Solution plan drafted.
- [x] Validation strategy documented.

---

## Phase III: Build

### Status

Phase III Complete — workflow implementation finished and pushed to my fork branch.

### Implementation Notes

I implemented the requested stale issue/PR cleanup workflow for Rafter CLI issue #35.

The implementation creates a new GitHub Actions workflow file:

```text
.github/workflows/stale.yml
```

The workflow uses `actions/stale@v9` and runs on a daily schedule. It also includes `workflow_dispatch` so maintainers can run the workflow manually if needed.

### Behavior Implemented

- Issues with no activity for 60 days are labeled `stale`.
- Stale issues with no activity for 14 more days are closed.
- Pull requests with no activity for 30 days are labeled `stale`.
- Pull requests are not automatically closed because the issue only requested stale labeling for PRs.
- Issues and PRs labeled `pinned` or `security` are exempt.
- The stale label is removed when new activity occurs.

### Files Modified

```text
.github/workflows/stale.yml
```

### Code Changes

| Item | Link |
|---|---|
| Branch | https://github.com/kietcoderlor/rafter-cli/tree/fix-35-stale-workflow |
| Issue | https://github.com/RafterSecurity/rafter-cli/issues/35 |
| Commit | https://github.com/Raftersecurity/rafter-cli/commit/7f095a7cfbbdb1b82f58d1373aa16142578c3306 |

### Testing / Validation

I validated the change by checking the workflow file and reviewing the diff.

Commands run:

```powershell
cd D:\codepath\rafter-cli
git status
git diff --check
git diff -- .github/workflows/stale.yml
```

### Validation Results

- The new workflow file exists at `.github/workflows/stale.yml`.
- The workflow uses `actions/stale@v9`.
- The issue stale and close timing matches issue #35.
- The PR stale timing matches issue #35.
- PRs are not auto-closed because the issue did not explicitly request PR auto-close.
- Exempt labels include `pinned` and `security`.
- The stale label is removed when new activity occurs.
- The change is limited to one workflow file.

### Challenges Faced

The main challenge was deciding whether stale pull requests should also be automatically closed. The issue explicitly says that inactive PRs should be labeled stale after 30 days, but it does not clearly request PR auto-close behavior. To avoid adding extra behavior beyond the issue requirements, I configured PRs to be labeled stale but not automatically closed.

### Phase III Checklist

- [x] Workflow file created.
- [x] `actions/stale@v9` configured.
- [x] Issue stale timing configured.
- [x] Issue close timing configured.
- [x] PR stale timing configured.
- [x] PR auto-close intentionally disabled.
- [x] Exempt labels configured.
- [x] Diff reviewed.
- [x] Branch pushed to fork.
- [ ] PR will be opened in Phase IV.
