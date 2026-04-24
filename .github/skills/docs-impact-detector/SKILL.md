---
name: docs-impact-detector
description: 'Detect code changes that require Docker Compose or Helm documentation updates in acs-deployment. Use when PRs modify compose files, Helm chart values, chart versions, image tags, env vars, search flavor behavior, ActiveMQ credentials, Kubernetes compatibility notes, or deployment examples.'
argument-hint: 'Optional: commit range or PR branch to analyze (for example: origin/master...HEAD)'
user-invocable: true
---

# Docs Impact Detector

Detects whether repository changes should trigger documentation updates, based on known ACS 26.1 impact patterns.

## When To Use
- Before opening or merging a PR.
- During release branch updates.
- When Docker Compose, Helm, chart versions, images, or environment variables change.
- When defaults, support status, or compatibility constraints change.

## Inputs
- Optional commit range (for example: origin/master...HEAD).
- If no range is provided, inspect unstaged plus staged local changes.

## Procedure
1. Collect changed files.
2. Prioritize files under Docker Compose and Helm paths.
3. Apply rules from [impact rules](./references/impact-rules.md).
4. Produce an impact report using [report template](./assets/report-template.md).
5. If any High severity finding exists, mark docs update as required before merge.

## File Scope
Primary source paths:
- docker-compose/compose.yaml
- docker-compose/community-compose.yaml
- docker-compose/23.N-compose.yaml
- docker-compose/25.N-compose.yaml
- docker-compose/7.4.N-compose.yaml
- docker-compose/pre-release-compose.yaml
- docker-compose/hxi-overrides.yaml
- helm/alfresco-content-services/Chart.yaml
- helm/alfresco-content-services/values.yaml
- helm/alfresco-content-services/*_values.yaml

Primary documentation paths likely impacted:
- docs/compose.md
- docs/docker-compose/README.md
- docs/helm.md
- docs/helm-charts.md
- docs/helm-deployment.md
- docs/helm/README.md
- docs/helm/eks-deployment.md
- docs/helm/examples/search-services.md
- docs/helm/examples/with-external-infrastructure.md
- docs/helm/examples/with-knowledge-retrieval.md

## Decision Logic
Mark docs impact as Required when one or more conditions are true:
- Image tag or version changed for repository, share, postgres, activemq, search, transform, or related components.
- New, removed, or renamed environment variable in deployable services.
- Default behavior changed (for example search flavor location or default value).
- Support statement changed (for example enterprise feature removal, community-only status).
- Chart metadata changed (chart version, appVersion, dependency versions).
- Compatibility/tested-platform statements changed (Kubernetes, EKS, database versions).
- Deployable file list changed (for example addition/removal of compose variants).

Mark docs impact as Verify when changes are indirect but can affect examples:
- Sub-chart dependency version updates without obvious values changes.
- Reference value changes in example-focused override files.

## Severity Guidance
- High: Breaking behavior, support removals, required auth/security changes, migration requirements.
- Medium: Default changes, mandatory parameter changes, version shifts affecting commands/examples.
- Low: Typo-level version mismatches, optional file list or table updates.

## Suggested Commands
Use fast searches and diffs. Prefer these patterns:
- git diff --name-only <range>
- git diff <range> -- docker-compose helm docs
- rg "image:|tag:|appVersion|version:|search\.flavor|ACTIVEMQ|messageBroker|Kubernetes|EKS|postgres|ALFRESCO_" docker-compose helm

## Output Requirements
The report must include:
- Changed source file
- Triggered rule
- Severity
- Documentation pages to update
- Concrete update instruction

If no rule triggers, explicitly state: No documentation-impacting changes detected by current rules.

## Notes
- This skill is tuned from ACS 26.1 findings and should be updated when new release patterns emerge.
- Keep detection rules aligned with release-to-release diffs and docs IA decisions.
