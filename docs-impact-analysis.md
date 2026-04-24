# ACS 26.1 Documentation Impact Analysis

**Date:** April 24, 2026  
**Repository:** `acs-deployment`  
**Documentation in scope:**
- [Alfresco Docker Compose Deployment](https://support.hyland.com/r/Current/Alfresco-Docker-Compose-Deployment/)
- [Install Using Helm — ACS 26.1](https://support.hyland.com/r/Alfresco/Alfresco-Content-Services/26.1/Alfresco-Content-Services/Install/Install-Using-Containers/Install-Using-Helm)

---

## Summary Table

| # | Area | Type | Severity | Documentation Section |
|---|---|---|---|---|
| 1 | Repository/Share/Postgres version references in examples | Update | Medium | Docker Compose — Install, AMP steps |
| 2 | Share image tag typo in env var example (`26.1.1` → `26.1.0`) | Fix | Low | Docker Compose — Environment Variables → Share |
| 3 | Live Ingester JAVA_OPTS image tag in example (`26.1` → `26.1.0`) | Fix | Low | Docker Compose — Configuring the Live Ingester |
| 4 | ActiveMQ 6.x credentials now mandatory in ACS 26.1 compose | Clarify | Medium | Docker Compose — Env vars for `transform-router`, `search`, `search-reindexing`, `alfresco` |
| 5 | `ALFRESCO_BULKINGESTER_ENDPOINT` undocumented in hxi-overrides | Add | Medium | Docker Compose — Configuring the Live Ingester |
| 6 | `ALFRESCO_REPOSITORY_VERSIONOVERRIDE: 25.1.0` stale in hxi-overrides.yaml | Fix | Medium | Docker Compose — Configuring the Live Ingester |
| 7 | `pre-release-compose.yaml` missing from Docker Compose files table | Add or acknowledge | Low | Docker Compose — Install → Downloading Files |
| 8 | **Solr/Search Services removed for Enterprise starting ACS 26.1** | **Breaking** | **High** | Helm — Considerations, search-services example, any Solr deployment steps |
| 9 | Helm chart version (10.3.2) and appVersion (26.1.0) in examples | Update | Medium | Helm — Prerequisites, EKS Deployment |
| 10 | ActiveMQ version bump (5.19.2 → 6.2.1) and authentication now always required | Update | High | Helm — External infrastructure, messageBroker config |
| 11 | Default `search.flavor` now set on `alfresco-repository`, not `global` | Update | Medium | Helm — Configuration, EKS Deployment, search examples |
| 12 | `alfresco-knowledge-retrieval` (hxi-connector) sub-chart version — verify docs | Verify | Low | Helm — `with-knowledge-retrieval.md` |
| 13 | PostgreSQL version bump (16.13 → 17.9) | Update | Low | Helm — supported platforms reference, EKS example |
| 14 | Kubernetes tested versions updated (k8s v1.34 / EKS v1.33) | Update | Medium | Helm — Prerequisites, Considerations |

---

## Part 1 — Docker Compose Documentation

### Finding 1 — Component Version Changes in Compose Files

**Severity:** Medium  
**Files:** `docker-compose/compose.yaml`, `docker-compose/25.N-compose.yaml`, `docker-compose/community-compose.yaml`

The following image versions changed between ACS 25.N and 26.1:

| Service | 25.N Tag | 26.1 Tag |
|---|---|---|
| `quay.io/alfresco/alfresco-content-repository` | `25.3.0` | `26.1.0` |
| `docker.io/alfresco/alfresco-content-repository-community` | prior tag | `26.1.0` |
| `quay.io/alfresco/alfresco-share` | `25.3.0` | `26.1.0` |
| `docker.io/alfresco/alfresco-share` (Community) | prior tag | `26.1.0` |
| `postgres` | `16.13` | `17.9` |

All other component images (ActiveMQ, Transform Router, Transform Core AIO, Shared File Store, Elasticsearch, Kibana, Live Indexing, Reindexing) remain at the same version between 25.N and 26.1.

**Required documentation updates:**
- The **"Applying AMP Files to the Alfresco Repository"** procedure:
  - Step 2 says to record the image name — the example should reference `quay.io/alfresco/alfresco-content-repository:26.1.0`.
  - Step 14 says to replace the image value — update the example from `25.3.0` to `26.1.0`.
- The **"Applying AMP Files to Alfresco Share"** procedure:
  - Steps 2 and 14: update the example from `quay.io/alfresco/alfresco-share:25.3.0` to `quay.io/alfresco/alfresco-share:26.1.0`.

---

### Finding 2 — Share Image Tag Typo in Environment Variables Documentation

**Severity:** Low  
**Location:** Docker Compose docs → Environment Variables → Alfresco Share (share)

The code example in the Share env vars section shows:

```yaml
share:
  image: quay.io/alfresco/alfresco-share:26.1.1
```

The actual tag in `compose.yaml` is `26.1.0`. The `1` on the patch segment is a typo.

**Required fix:** Change `26.1.1` → `26.1.0` in the Share env vars code example.

---

### Finding 3 — Live Ingester JAVA_OPTS Example Uses Incomplete Version Tag

**Severity:** Low  
**Location:** Docker Compose docs → Configuring the Deployment → Configuring the Live Ingester → Step 5

The example JAVA_OPTS snippet in the Live Ingester configuration section shows:

```yaml
alfresco:
  image: quay.io/alfresco/alfresco-content-repository:26.1
```

The version tag `26.1` is missing the patch segment. The correct tag is `26.1.0`.

**Required fix:** Change `:26.1` → `:26.1.0` in the Live Ingester step 5 code example.

---

### Finding 4 — ActiveMQ 6.x Credentials Now Mandatory in ACS 26.1 Enterprise Compose

**Severity:** Medium  
**Files:** `docker-compose/compose.yaml` vs `docker-compose/25.N-compose.yaml`

ACS 26.1 ships with **ActiveMQ 6.2.1** as the default broker, which requires authentication. The `compose.yaml` for ACS 26.1 explicitly includes the following credentials that were missing from `25.N-compose.yaml`:

| Service | Variable Added | Value |
|---|---|---|
| `alfresco` (JAVA_OPTS) | `-Dmessaging.broker.username=admin` | `admin` |
| `alfresco` (JAVA_OPTS) | `-Dmessaging.broker.password=admin` | `admin` |
| `transform-router` | `ACTIVEMQ_USER` | `admin` |
| `transform-router` | `ACTIVEMQ_PASSWORD` | `admin` |
| `search` (live indexing) | `SPRING_ACTIVEMQ_USER` | `admin` |
| `search` (live indexing) | `SPRING_ACTIVEMQ_PASSWORD` | `admin` |
| `search-reindexing` | `SPRING_ACTIVEMQ_USER` | `admin` |
| `search-reindexing` | `SPRING_ACTIVEMQ_PASSWORD` | `admin` |

The current env var reference tables document these with the qualifier: *"Only required when using ActiveMQ 6.x or later."*

**Required documentation updates:**
- Since ACS 26.1 ships with ActiveMQ 6.x by default, the qualifier language should be updated to state these credentials **are required by default** in ACS 26.1 deployments.
- Consider adding a note to the ActiveMQ section or the install prerequisites that ActiveMQ 6.x is the default and authentication is enabled.

---

### Finding 5 — `ALFRESCO_BULKINGESTER_ENDPOINT` Not Documented

**Severity:** Medium  
**File:** `docker-compose/hxi-overrides.yaml`

The `knowledge-retrieval` service in `hxi-overrides.yaml` includes the following environment variable that is not present in the Live Ingester environment variable tables in the documentation:

```yaml
ALFRESCO_BULKINGESTER_ENDPOINT: activemq:queue:bulk-ingester-events
```

**Required documentation update:** Add `ALFRESCO_BULKINGESTER_ENDPOINT` to the Live Ingester / Knowledge Retrieval environment variable reference table, with a description of its purpose (specifying the ActiveMQ destination queue for bulk ingestion events).

---

### Finding 6 — Stale ACS Version in `hxi-overrides.yaml`

**Severity:** Medium  
**File:** `docker-compose/hxi-overrides.yaml`

The `knowledge-retrieval` service contains:

```yaml
ALFRESCO_REPOSITORY_VERSIONOVERRIDE: 25.1.0
```

This env var hardcodes the ACS repository version to `25.1.0`, which is stale for an ACS 26.1 deployment.

**Required fix:**
- Update `ALFRESCO_REPOSITORY_VERSIONOVERRIDE` to `26.1.0` in `hxi-overrides.yaml`.
- If this variable is documented in the Live Ingester configuration steps, the example value should also be updated.

---

### Finding 7 — `pre-release-compose.yaml` Missing from Available Files Table

**Severity:** Low  
**Location:** Docker Compose docs → Installing Content Services → Downloading the Docker Compose Files

The documentation table of available Docker Compose files lists:

| File | Description |
|---|---|
| `compose.yaml` | Latest Content Services version |
| `25.N-compose.yaml` | 25.x release family |
| `23.N-compose.yaml` | 23.x release family |
| `7.4.N-compose.yaml` | 7.4.x release family |
| `community-compose.yaml` | Community Edition |

The file `pre-release-compose.yaml` exists in the repository (`docker-compose/pre-release-compose.yaml`) but is not listed in this table.

**Required documentation update:** Either add `pre-release-compose.yaml` to the table with a note that it deploys pre-release component versions for testing, or add a note below the table acknowledging its existence and purpose so users who encounter it understand what it is.

---

## Part 2 — Helm Chart Documentation

### Finding 8 — BREAKING: Alfresco Search Services (Solr) No Longer Supported for Enterprise

**Severity:** High — Breaking Change  
**File:** `helm/alfresco-content-services/values.yaml`

```yaml
alfresco-search:
  # -- Search Services (`alfresco-search`) is no longer supported for Enterprise
  # edition starting with ACS v26
  enabled: false
```

Starting with **ACS 26.1**, Solr-based Search Services (`alfresco-search`) is **disabled by default and unsupported for Enterprise Edition**. Elasticsearch-based Search Enterprise (`alfresco-search-enterprise`) is now the **only supported search option for Enterprise**.

The default repository configuration explicitly sets:
```yaml
alfresco-repository:
  configuration:
    search:
      flavor: elasticsearch
```

**Required documentation updates:**
- **Helm — Considerations page:** Add a clear statement that Search Services (Solr) is no longer supported for Enterprise Edition starting with ACS 26.1. Only `alfresco-search-enterprise` (Elasticsearch) is supported.
- **`docs/helm/examples/search-services.md`:** Mark this example as **Community Edition only** or remove it for Enterprise-targeted documentation.
- **Any deployment guide steps** that mention enabling Solr for Enterprise must be updated or removed.
- The `global.search.flavor: solr6` configuration option should be clearly marked as Community-only in the Helm docs.
- Add a migration note for customers upgrading from ACS 25.x who were using Solr with Enterprise — they must migrate to Search Enterprise before upgrading to 26.1.

---

### Finding 9 — Helm Chart Version and appVersion in Examples

**Severity:** Medium  
**File:** `helm/alfresco-content-services/Chart.yaml`

| Field | Value |
|---|---|
| Chart version | `10.3.2` |
| `appVersion` | `26.1.0` |

Any Helm install commands in the documentation that reference a specific chart version number need to be updated. For example, commands of the form:

```bash
helm install acs alfresco/alfresco-content-services --version X.X.X
```

**Required documentation updates:**
- Update all `--version` references to chart version `10.3.2` where a specific version is cited.
- Update any `appVersion` references to `26.1.0`.

---

### Finding 10 — ActiveMQ Version Bump and Authentication Always Required

**Severity:** High  
**File:** `helm/alfresco-content-services/values.yaml`

ActiveMQ images version changed from `5.19.2-jre17-rockylinux8` to **`6.2.1-jre17-rockylinux8`**.

The `values.yaml` now explicitly defines admin credentials in the `activemq` section:

```yaml
activemq:
  image:
    tag: 6.2.1-jre17-rockylinux8
  adminUser:
    user: admin
    password: admin
    existingSecretName: *acs_messageBroker_secretName
```

All sub-charts that connect to the broker — `alfresco-transform-service`, `alfresco-search-enterprise`, `alfresco-sync-service`, `alfresco-audit-storage`, `alfresco-knowledge-retrieval` — propagate credentials via the `messageBroker.existingSecret` pattern.

**Required documentation updates:**
- **Helm — External Infrastructure example** (`docs/helm/examples/with-external-infrastructure.md`): Update to reflect that external broker configuration must include credentials and the `messageBroker.existingSecretName` approach is the recommended pattern.
- **Helm — Prerequisites or Considerations:** Add a note that ActiveMQ 6.x is the default and requires broker authentication. All connected services must be configured with matching credentials.
- Any documentation showing ActiveMQ 5.x configuration examples should be updated to ActiveMQ 6.x equivalents.

---

### Finding 11 — Default Search Flavor Now Set on `alfresco-repository`, Not `global`

**Severity:** Medium  
**File:** `helm/alfresco-content-services/values.yaml`

Previously, the search flavor was configured via:
```yaml
global:
  search:
    flavor: elasticsearch
```

In ACS 26.1, the default is now set directly on the repository sub-chart:
```yaml
alfresco-repository:
  configuration:
    search:
      flavor: elasticsearch
```

**Required documentation updates:**
- Any Helm configuration guide, EKS deployment procedure, or example that instructs users to set `global.search.flavor` should be updated to use `alfresco-repository.configuration.search.flavor`.
- The `global.search.url`, `global.search.securecomms`, and related properties remain relevant for external search service integration — clarify which properties belong to `global` vs `alfresco-repository.configuration.search`.

---

### Finding 12 — Knowledge Retrieval (HxI Connector) Sub-chart Version

**Severity:** Low  
**File:** `helm/alfresco-content-services/Chart.yaml`

```yaml
- name: alfresco-connector-hxi
  alias: alfresco-knowledge-retrieval
  version: 0.5.0
```

**Required action:** Verify the `docs/helm/examples/with-knowledge-retrieval.md` example file reflects the current chart version and any configuration changes introduced in `alfresco-connector-hxi` version `0.5.0`. If the configuration interface changed since the last documented version, update accordingly.

---

### Finding 13 — PostgreSQL Version Bump (16.13 → 17.9)

**Severity:** Low  
**Files:** `docker-compose/compose.yaml`, `docker-compose/community-compose.yaml`

The embedded PostgreSQL image version changed from `16.13` to `17.9`.

**Required documentation updates:**
- Update the supported platforms reference if it specifies the embedded PostgreSQL version bundled with the deployment files.
- Update any code examples in the EKS or Kind deployment guides that specify a PostgreSQL version.

---

### Finding 14 — Kubernetes Tested Versions Updated

**Severity:** Medium  
**Source:** `alfresco.github.io/acs-deployment` (alfresco.github.io badge — `k8s version: v1.34`, EKS tested on `v1.33`)

**Required documentation updates:**
- **Helm — Considerations:** The note referencing supported Kubernetes versions should point readers to the ACS Deployment docs site, which now reflects k8s v1.34 / EKS v1.33.
- **Helm — Prerequisites:** Update or verify the supported Kubernetes version call-out.
- **EKS Deployment section:** Verify that the documented `--node-type` and cluster setup steps are still valid for EKS v1.33.

---

## Appendix — File Reference

| Repository File | Relevance |
|---|---|
| `docker-compose/compose.yaml` | ACS 26.1 Enterprise baseline compose |
| `docker-compose/25.N-compose.yaml` | ACS 25.x comparison baseline |
| `docker-compose/community-compose.yaml` | ACS 26.1 Community compose |
| `docker-compose/hxi-overrides.yaml` | Knowledge Retrieval / Live Ingester override |
| `docker-compose/pre-release-compose.yaml` | Pre-release component versions |
| `helm/alfresco-content-services/Chart.yaml` | Helm chart version and sub-chart dependencies |
| `helm/alfresco-content-services/values.yaml` | Default Helm values for ACS 26.1 |
| `helm/alfresco-content-services/25.N_values.yaml` | ACS 25.x Helm values comparison baseline |
