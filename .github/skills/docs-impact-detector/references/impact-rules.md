# Impact Rules For ACS Deployment Docs

Use these rules to map code changes to likely documentation updates.

## Rule 1: Core Compose Image Version Drift
Trigger:
- Changes to repository/share/postgres image tags in compose files.

Detect in:
- docker-compose/compose.yaml
- docker-compose/community-compose.yaml
- docker-compose/*-compose.yaml

Docs likely impacted:
- docs/compose.md
- docs/docker-compose/README.md
- AMP procedures and image examples

Severity:
- Medium

## Rule 2: Example Version Mismatch Or Typo
Trigger:
- A docs example references a patch tag not present in source compose values.

Detect by comparing:
- docs examples vs docker-compose/*.yaml actual tags

Docs likely impacted:
- docs/compose.md
- docs/docker-compose/README.md

Severity:
- Low

## Rule 3: Live Ingester Tag And Endpoint Changes
Trigger:
- Changes to live ingester related repository tag snippets or hxi endpoint variables.
- Added/removed variables such as ALFRESCO_BULKINGESTER_ENDPOINT.

Detect in:
- docker-compose/hxi-overrides.yaml
- docker-compose/compose.yaml

Docs likely impacted:
- docs/compose.md
- docs/docker-compose/examples/customisation-guidelines.md

Severity:
- Medium

## Rule 4: ActiveMQ Authentication Requirement Changes
Trigger:
- ActiveMQ major version changes.
- Broker credentials added/removed/renamed in compose or Helm values.

Detect in:
- docker-compose/compose.yaml
- helm/alfresco-content-services/values.yaml

Look for keys:
- ACTIVEMQ_USER
- ACTIVEMQ_PASSWORD
- SPRING_ACTIVEMQ_USER
- SPRING_ACTIVEMQ_PASSWORD
- messageBroker.existingSecretName
- adminUser.user
- adminUser.password

Docs likely impacted:
- docs/compose.md
- docs/helm/examples/with-external-infrastructure.md
- docs/helm-deployment.md

Severity:
- High

## Rule 5: Compose File Matrix Changes
Trigger:
- New or removed compose variant files (for example pre-release compose).

Detect in:
- docker-compose/ directory file list

Docs likely impacted:
- docs/compose.md
- docs/docker-compose/README.md

Severity:
- Low

## Rule 6: Enterprise Search Support Policy Changes
Trigger:
- Solr/alfresco-search enablement changes.
- Comments or values indicating enterprise support removal.

Detect in:
- helm/alfresco-content-services/values.yaml

Docs likely impacted:
- docs/helm.md
- docs/helm/examples/search-services.md
- docs/helm-guides.md

Severity:
- High

## Rule 7: Search Flavor Configuration Path Changes
Trigger:
- Default path/value moves between global.search.flavor and alfresco-repository.configuration.search.flavor.

Detect in:
- helm/alfresco-content-services/values.yaml
- helm/alfresco-content-services/*_values.yaml

Docs likely impacted:
- docs/helm-deployment.md
- docs/helm/eks-deployment.md
- docs/helm/examples/search-services.md

Severity:
- Medium

## Rule 8: Chart Metadata And Dependency Version Changes
Trigger:
- Chart version, appVersion, or dependency versions change.

Detect in:
- helm/alfresco-content-services/Chart.yaml

Docs likely impacted:
- docs/helm.md
- docs/helm-charts.md
- docs/helm/eks-deployment.md
- docs/helm/examples/with-knowledge-retrieval.md

Severity:
- Medium for chart/app version
- Low for dependency verification only

## Rule 9: Platform Compatibility Version Changes
Trigger:
- Kubernetes, EKS, or Postgres version updates in deployment defaults.

Detect in:
- docker-compose/*.yaml
- helm/alfresco-content-services/values.yaml
- release metadata and badges if maintained in repo

Docs likely impacted:
- docs/helm.md
- docs/helm-deployment.md
- docs/helm/eks-deployment.md
- docs/compose.md

Severity:
- Medium for Kubernetes/EKS
- Low for embedded Postgres version callouts

## Rule 10: Breaking Change Migration Requirement
Trigger:
- A previously supported enterprise path becomes unsupported.

Detect via:
- support comments, disabled defaults, or explicit notes in values/comments

Docs likely impacted:
- docs/helm.md
- docs/helm-guides.md
- upgrade and migration sections

Severity:
- High

## Rule 11: Secret Wiring Pattern Changes
Trigger:
- Changes to existingSecretName/existingSecret wiring for broker or dependent services.

Detect in:
- helm/alfresco-content-services/values.yaml
- dependent chart value blocks

Docs likely impacted:
- docs/helm/examples/with-external-infrastructure.md
- docs/helm-deployment.md

Severity:
- Medium

## Rule 12: Docs Drift Guard
Trigger:
- Any source change that modifies defaults/examples without matching docs changes in the same PR.

Detect by comparing:
- changed source files under docker-compose/ or helm/
- changed docs files under docs/

Severity:
- Medium by default; escalate if Rules 4, 6, or 10 also trigger

Recommendation:
- Require explicit reviewer note if no docs updates are included.
