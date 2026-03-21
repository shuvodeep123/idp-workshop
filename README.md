# Introduction

Starter kit for a Harness IDP Workshop

---

## What is this repository?

This repository is a **starter kit** for a **Harness IDP (Internal Developer Portal) Workshop**. It contains configuration files that teach you how to register software components, APIs, and self-service workflows inside Harness IDP — which is built on top of [Backstage](https://backstage.io/), an open-source developer portal platform originally created by Spotify.

Think of Harness IDP as a **central catalog** where every service, API, pipeline, and team in your organization is listed, documented, and discoverable — like an internal "Google" for your engineering platform.

---

## Repository Structure

```
idp-workshop/
├── README.md                          # This file — project documentation
├── api/                               # API descriptor files
│   ├── petstore-api.yaml              # Backstage API entity for the Petstore REST API
│   ├── petstore.oas.yaml              # Full OpenAPI 3.x spec for the Petstore API
│   └── spotify.yaml                  # Backstage API entity for the Spotify Web API
├── components/                        # Service/component descriptor files
│   ├── catalog-info.yaml              # Main service component definition
│   ├── catalog-info-dependent.yaml    # Variant service component (with dependency placeholders)
│   └── system.yaml                   # System entity that groups related components
├── onboard-new-app.yaml               # Backstage Scaffolder template — provisions a new app via pipeline
├── bonus-onboarding-template.yaml     # Bonus Scaffolder template — creates a cookiecutter data science app
└── .harness/                          # Harness CI/CD pipeline configuration
    ├── pipelines/
    │   └── idp-workshop-*.yaml        # CI pipeline definition for this repo
    ├── Runtime_OWASP_CVEs.rego        # OPA policy to block builds with critical Node.js vulnerabilities
    └── *-trigger-input-set-*.yaml     # Input sets for PR and push pipeline triggers
```

---

## File-by-File Explanation

### `api/petstore-api.yaml` — Backstage API Entity (Petstore)

**What it does:** Registers the well-known [Swagger Petstore](https://github.com/swagger-api/swagger-petstore) REST API into the Harness IDP catalog so teams can discover and understand it.

**Technical details:**
- `apiVersion: backstage.io/v1alpha1` — Uses the Backstage catalog schema version 1 alpha.
- `kind: API` — Declares this YAML as an **API entity** in the catalog (not a service or system).
- `spec.type: openapi` — Tells Backstage the API is described using an **OpenAPI** specification.
- `spec.definition.$text: ./petstore.oas.yaml` — Points to the actual OpenAPI spec file stored alongside it. Backstage fetches and renders this spec as interactive API documentation.
- `spec.lifecycle: dev` — Marks this API as being in the **development** lifecycle stage.
- `spec.owner: TeamA` — Assigns ownership of this API to **TeamA** in the catalog.

---

### `api/petstore.oas.yaml` — OpenAPI Specification (Petstore)

**What it does:** This is the full **OpenAPI 3.x specification** for the Petstore API. It describes every HTTP endpoint, request/response schema, authentication method, and data model for the API.

**Technical details:**
- Written in **YAML** following the [OpenAPI Specification](https://swagger.io/specification/) standard.
- Used by tools like Swagger UI, Backstage, and API gateways to auto-generate documentation, client SDKs, and mock servers.
- Referenced by `petstore-api.yaml` via the `$text` placeholder, so Backstage can inline and render the full spec.

---

### `api/spotify.yaml` — Backstage API Entity (Spotify)

**What it does:** Registers the public [Spotify Web API](https://developer.spotify.com/documentation/web-api/) into the IDP catalog so developers can see what external APIs their services consume.

**Technical details:**
- `spec.type: openapi` — Again an OpenAPI-based API.
- `spec.definition.$text` — Points to a remote URL on GitHub (the APIs-guru OpenAPI directory) rather than a local file. Backstage fetches the spec at catalog-import time.
- `annotations.backstage.io/definition-at-location` — An older, deprecated annotation that previously told Backstage where to find the spec. The `$text` placeholder approach in `spec.definition` is the modern replacement.
- `spec.owner: TeamB` — Owned by **TeamB**.

---

### `components/catalog-info.yaml` — Service Component (Main)

**What it does:** Registers a sample **microservice** into the IDP catalog. This is the central file workshop participants edit to add their own service.

**Technical details:**
- `kind: Component` — Declares a **Component entity**, representing a deployable software piece (microservice, library, website, etc.).
- `spec.type: service` — This component is specifically a backend **service**.
- `spec.lifecycle: experimental` — Lifecycle flag indicating the service is still in early development.
- `spec.consumesApis: [spotify]` — Declares that this service **calls** the Spotify API (creates a dependency edge in the catalog graph).
- `spec.providesApis: [petstore]` — Declares that this service **exposes** the Petstore API (links the component to the API entity).
- `annotations`:
  - `github.com/project-slug` — Links to the GitHub repository for source code browsing.
  - `backstage.io/techdocs-ref: dir:../` — Enables **TechDocs** (MkDocs-based documentation) by pointing to the root of the repo.
  - `lighthouse.com/website-url` — Used by the **Lighthouse** plugin to run web performance audits.
  - Commented-out annotations show optional integrations: **Harness CI/CD pipelines**, **IACM (Infrastructure as Code Management)**, **Feature Flags**, **Kubernetes**, **PagerDuty**, and **Jira** plugins.
- `spec.owner: change-value` — Placeholder that workshop participants replace with their actual team/group name.

---

### `components/catalog-info-dependent.yaml` — Service Component (Dependency Variant)

**What it does:** A secondary service definition used in the workshop to demonstrate **component dependencies** — how one service can declare it depends on another service.

**Technical details:**
- Nearly identical to `catalog-info.yaml` but named `yournamehere-service-dependency`.
- Contains commented-out `spec.dependsOn` and `spec.system` fields. When uncommented, these fields create **dependency relationships** in the catalog, e.g., "this service depends on `yournamehere-service`" and "this service belongs to `yournamehere-system`".
- These relationships are visualized as a **dependency graph** in the IDP UI.

---

### `components/system.yaml` — System Entity

**What it does:** Defines a **System** — a logical grouping of related components and APIs. In Backstage, a System is like a "product" or "bounded context" that owns multiple microservices.

**Technical details:**
- `kind: System` — Backstage system entity.
- Components opt-in to this system by adding `spec.system: yournamehere-system` in their own YAML (currently commented out in `catalog-info.yaml`).
- `spec.owner: TeamA` — This system is owned by **TeamA**.

---

### `onboard-new-app.yaml` — Scaffolder Template (Self-Service App Provisioning)

**What it does:** A **Backstage Scaffolder Template** (also called a "Workflow" in Harness IDP) that gives developers a self-service form in the IDP UI to provision a brand-new application — including creating a GitHub repository — by triggering a Harness pipeline.

**Technical details:**
- `kind: Template` / `apiVersion: scaffolder.backstage.io/v1beta3` — Backstage scaffolder schema.
- `spec.parameters` — Defines the **UI form fields** presented to the developer:
  - `github_repo_name`, `github_repo_description` — Standard text inputs.
  - `github_repo_owner` — Uses `ui:field: OwnerPicker`, a Backstage built-in field that renders a dropdown of catalog groups/users.
  - `token` — Uses `ui:field: HarnessAuthToken` and `ui:widget: password` to securely capture the developer's Harness API token at runtime.
- `spec.steps` — Defines what happens when the form is submitted:
  - The single step uses the `trigger:harness-custom-pipeline` action, which calls the **Harness Pipeline API** to kick off a pipeline at the given URL with the form values as pipeline inputs (`inputset`).
- `spec.output.links` — After the pipeline starts, the UI displays a link to the running pipeline, so developers can track progress.

---

### `bonus-onboarding-template.yaml` — Scaffolder Template (Data Science App)

**What it does:** A bonus Scaffolder Template for creating a new **cookiecutter data science project** by triggering a Harness pipeline. Demonstrates a multi-step form with two separate parameter pages.

**Technical details:**
- Two `parameters` pages (multi-step wizard in the UI):
  1. **Repository onboarding details** — collects `project_name` and `feature_branch` (the initial working branch for the new repo).
  2. **Service Infrastructure Details** — collects `owner` (via `OwnerPicker`) and `token` (via `HarnessAuthToken`).
- The `steps` block triggers a Harness pipeline just like in `onboard-new-app.yaml`, but the pipeline URL is a placeholder (`<input onboarding pipeline URL>`) that workshop participants replace with their own pipeline.
- This template demonstrates how to structure **multi-page self-service forms** for more complex onboarding workflows.

---

### `.harness/pipelines/idp-workshop-*.yaml` — Harness CI Pipeline

**What it does:** Defines a **Harness CI pipeline** for building this repository. It is automatically triggered on pull requests and pushes.

**Technical details:**
- `pipeline.stages[].type: CI` — This is a **Continuous Integration** stage.
- `spec.cloneCodebase: true` — The pipeline clones this Git repository before running any steps.
- `spec.execution.steps` — The single step runs a shell command (`echo "Welcome to Harness CI"`) as a demonstration. In a real pipeline, this would run tests, build Docker images, run security scans, etc.
- `spec.platform.os: Linux / arch: Amd64` — The pipeline runs on a **Linux AMD64** cloud runner managed by Harness.
- `spec.runtime.type: Cloud` — Uses **Harness Cloud** (ephemeral, managed build infrastructure) instead of a self-hosted build runner.
- `properties.ci.codebase.connectorRef: account.idp` — Uses a pre-configured **Git connector** called `idp` to authenticate with the source code repository.
- `properties.ci.codebase.build: <+input>` — The specific branch/PR/tag to build is passed in as a **runtime input**, populated by the trigger input sets.

---

### `.harness/Runtime_OWASP_CVEs.rego` — OPA Security Policy

**What it does:** An **Open Policy Agent (OPA)** policy written in the **Rego** policy language. It enforces a security gate: if any Node.js open-source dependency has a **critical CVE (Common Vulnerability and Exposure)**, the pipeline is denied from proceeding.

**Technical details:**
- `package pipeline_environment` — Declares the OPA package namespace.
- `deny[...]` — OPA's standard `deny` rule; if this rule evaluates to true for any input, the policy denies the action (blocks the pipeline).
- `input.NODE_OSS_CRITICAL_COUNT` — Reads a variable from the pipeline's runtime context that holds the count of critical Node.js OSS vulnerabilities found during an SCA (Software Composition Analysis) scan.
- The rule triggers if `NODE_OSS_CRITICAL_COUNT != 0` — i.e., even a single critical vulnerability causes the policy to fail and the build to be blocked.
- This is an example of **Policy-as-Code (PaC)** integrated into the CI/CD pipeline to enforce security compliance automatically.

---

### `.harness/*-trigger-input-set-*.yaml` — Pipeline Trigger Input Sets

**What it does:** These YAML files provide the **input values** passed to the CI pipeline when it is triggered by a GitHub event (pull request or push).

**Technical details:**
- Harness **Input Sets** separate the pipeline definition from the runtime variables, making pipelines reusable across different contexts.
- The **PR trigger input set** tells the pipeline to build the pull request's head branch/commit.
- The **push trigger input set** tells the pipeline to build the branch that was just pushed to.
- Both are connected to the pipeline via the `codebase.build: <+input>` placeholder in the pipeline YAML.

---

## How It All Fits Together

```
Developer fills out        Scaffolder Template        Harness Pipeline
a self-service form   →   (onboard-new-app.yaml)  →  creates GitHub repo
in Harness IDP UI                                      + registers catalog
                                                         entry
        ↓
Harness IDP Catalog shows:
  ┌─────────────────────────────────────────────┐
  │  System: yournamehere-system                 │
  │    └─ Component: yournamehere-service         │
  │         ├─ provides API: petstore             │
  │         └─ consumes API: spotify              │
  └─────────────────────────────────────────────┘
        ↓
Any code push to this repo triggers the
Harness CI pipeline (.harness/pipelines/*.yaml)
which is gated by the OPA security policy
(.harness/Runtime_OWASP_CVEs.rego)
```

In summary, this repository demonstrates the **three core pillars** of Harness IDP:
1. **Software Catalog** — YAML-defined entities (`Component`, `API`, `System`) that describe what you have.
2. **Self-Service Workflows** — Scaffolder templates that let developers provision infrastructure/repos without needing DevOps help.
3. **CI/CD Integration** — Harness pipelines that build, test, and enforce security policies automatically.