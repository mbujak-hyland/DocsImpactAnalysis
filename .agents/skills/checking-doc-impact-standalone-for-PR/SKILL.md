---
name: checking-doc-impact
description: 'Analyze the diff of a specific merged PR in a specific GitHub repository to determine whether a documentation update is required. Outputs a structured verdict used by a downstream workflow to notify a technical writer.'
argument-hint: 'PR number and GitHub repository (e.g. 1234 alfresco/acs-deployment)'
---

# PR Documentation Impact Check

Analyze the diff of a specific merged PR to determine whether a documentation
update is required. This skill produces a single structured verdict consumed
by an automated workflow to notify a technical writer when action is needed.

This skill does **not** locate doc sections, draft content, or invoke
downstream skills.

## When to Use

- A PR has been merged and the workflow needs to know if a technical writer
  must be notified
- When asked "does this merged PR require a documentation update?"

## Inputs

| Input | Required | Description |
|---|---|---|
| PR number | Yes | The merged PR number |
| Repository | Yes | Full `owner/repo` slug (e.g. `alfresco/acs-deployment`) |

---

## Procedure

### Step 1: Fetch the Merged PR Diff

Use the GitHub CLI to fetch the diff for the specified PR and repository:

```bash
gh pr diff <PR_NUMBER> --repo <owner>/<repo>
```

Also fetch the PR metadata to include in the verdict:

```bash
gh pr view <PR_NUMBER> --repo <owner>/<repo> --json number,title,url,mergedAt,author,baseRefName
```

Read the full diff. For each changed file, note:
- The file path and file type
- Lines added (`+`) and removed (`-`)
- The nature of the change (new parameter, renamed key, new resource,
  removed option, changed default, new behavior, etc.)

---

### Step 2: Classify Each Changed File

For each changed file, apply the rules below to determine whether it is
doc-impacting.

**Always doc-impacting:**
- New configuration keys or parameters (any `values.yaml`, config file,
  environment variable, CLI flag, API field)
- Removed or renamed configuration keys (breaking change)
- Changed default values that alter behavior
- New Kubernetes resource types (RBAC, PVC, Ingress, CRD, HPA)
- New or changed ingress paths, ports, or hostnames
- New dependencies or prerequisites
- New operational procedures (hooks, jobs, upgrade steps)
- New features or capabilities exposed to users or operators

**Never doc-impacting:**
- Version-only bumps (`version`, `appVersion`, dependency version numbers)
- Internal refactors with no behavioral or interface change
- CI/CD pipelines, build scripts, release automation, or developer tooling
- Security-only fixes (CVE remediations) with no user-facing or
  operator-facing configuration or behavior change
- Test code, test data, or test harness changes with no production behavior
  change
- Whitespace, comment, or formatting-only changes
- Auto-generated files (`Chart.lock`, generated READMEs)

**Ambiguous (treat as doc-impacting when in doubt):**
- Default value changes where deployment impact is unclear
- Internal variable renames that may surface in error messages or logs
- Changes inside conditionals where the condition is user-controllable

---

### Step 3: Produce the Verdict

#### If NO documentation update is required

```
## Documentation Verdict: Not Required

**PR:** <PR_NUMBER> — <PR title>
**Repository:** <owner/repo>
**PR URL:** <url>
**Merged at:** <mergedAt>
**Author:** <author login>

**Reason:** <One or two sentences citing the specific heuristic that applies,
e.g. "All changes are version bumps with no behavioral change" or "Changes
are limited to CI pipeline configuration with no user-facing impact.">

**Files reviewed:** <count> files changed, none doc-impacting
```

#### If a documentation update IS required

```
## Documentation Verdict: Required

**PR:** <PR_NUMBER> — <PR title>
**Repository:** <owner/repo>
**PR URL:** <url>
**Merged at:** <mergedAt>
**Author:** <author login>
**Breaking changes:** Yes / No

### Findings

**[F1] <short description of the change>**
- File: `<changed file path>`
- Change type: New parameter / Removed key / Changed default / New feature / Breaking change / etc.
- Audience: User-facing / Operator-facing / Maintainer-facing
- Breaking: Yes / No
- Summary: <One or two sentences on what changed and why it needs documentation.>

(Repeat for each doc-impacting change. List breaking changes first.)

### Ambiguous Changes

**[A1] <short description>**
- File: `<file path>`
- Why ambiguous: <one-line explanation>
- Treated as: Doc-impacting (conservative default)
```

---

## Quality Checks Before Finishing

- Verdict is one of exactly two values: `Required` or `Not Required`
- PR number, repository, URL, merge timestamp, and author are present in all verdicts
- If not required: a specific heuristic reason is given
- If required: every doc-impacting file has a finding block
- Breaking changes appear first in the findings list
- Ambiguous items are listed separately and treated as doc-impacting by default
- Version bumps, whitespace changes, CI changes, and test-only changes are absent from findings
- No section targets, page URLs, or draft content are included
