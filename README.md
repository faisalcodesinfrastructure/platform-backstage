# platform-backstage

> **Platform Engineering Golden Path — Workshop Series**
> Internal Developer Portal · Phase 5

---

## What is this repository?

This repository contains the configuration, templates, and documentation
for the **Backstage Internal Developer Portal (IDP)** — the front door
of the entire golden path.

Backstage is what developers actually interact with. Instead of reading
wiki pages about how to set up a service, opening tickets to request
infrastructure, or asking a platform engineer for help — a developer
opens Backstage, fills out a form, and gets a fully wired production-
ready service in minutes.

This repo is the configuration layer that makes that experience possible.

---

## Workshop context

This is one of three repositories in the Platform Engineering Golden Path workshop:

| Repo | Purpose |
|------|---------|
| `platform-infra` | Cluster, IaC, GitOps manifests — platform team |
| `golden-app` | The sample app developers receive — airline domain |
| **platform-backstage** (this repo) | Backstage IDP — templates, catalog, docs |

This repo is built entirely in **Phase 5**, the final phase of the workshop.
By the time you get here, the cluster is running, the airline app is
deployed, and CI/CD is working. Phase 5 wraps all of that in a portal
so a second developer can reproduce the entire thing from a form.

---

## What Backstage provides

### Software Catalog
A searchable, browsable registry of every service, library, pipeline,
and team in the organisation. Each entry is defined by a
`catalog-info.yaml` file that lives alongside the code it describes.

In this workshop, the `golden-app` airline service is registered in
the catalog so you can see its health, documentation, deployment
status, and ownership from one place.

### Scaffolder (golden path templates)
Form-driven wizards that create a new repository from a template.
The developer fills in fields (app name, description, owner), clicks
"Create", and Backstage:
- Creates the GitHub repo
- Copies and renders the skeleton files
- Registers the new service in the catalog
- Triggers the first CI run

The template in this repo produces a repo identical to `golden-app`
with the developer's chosen name substituted throughout.

### TechDocs
Markdown documentation that lives alongside code (in a `docs/` folder)
and renders as searchable, versioned pages inside the portal.
No separate documentation site to maintain.

### Flux plugin (Phase 5)
Shows the live deployment status of each catalog service — whether Flux
has synced it, when it last reconciled, and whether it is healthy —
without leaving the portal.

---

## Technology stack

| Component | Tool | Notes |
|-----------|------|-------|
| Portal runtime | [Backstage](https://backstage.io) (Node.js) | CNCF project, v1.x |
| Package manager | npm | Node 20 LTS required |
| Scaffolder integration | GitHub App / PAT | Creates repos on behalf of developers |
| Catalog source | GitHub (this repo + app repos) | catalog-info.yaml auto-discovery |
| Documentation | TechDocs (MkDocs under the hood) | Markdown → rendered portal pages |
| CD visibility | Flux Backstage plugin | Shows sync status per service |

---

## Repository structure

```
platform-backstage/
│
├── README.md                        ← you are here
│
├── app-config.yaml                  # Phase 5: main Backstage configuration
│                                    #   GitHub auth, catalog locations,
│                                    #   plugin config, integrations
│
├── app-config.local.yaml            # Local secrets — NEVER committed
│                                    #   GitHub token, auth secrets
│                                    #   (listed in .gitignore)
│
├── catalog/
│   └── catalog-info.yaml            # Phase 5: registers golden-app in catalog
│                                    #   Component kind, owner, links, techdocs
│
├── templates/
│   └── go-app-template/
│       ├── template.yaml            # Phase 5: Scaffolder template definition
│       │                            #   Form fields, automation steps,
│       │                            #   what gets created and registered
│       └── skeleton/                # Phase 5: files copied into new repos
│           ├── cmd/server/main.go
│           ├── Dockerfile
│           ├── go.mod
│           ├── helm/${{values.appName}}/
│           │   ├── Chart.yaml
│           │   └── values.yaml
│           └── .github/workflows/ci.yaml
│
└── techdocs/
    └── docs/
        ├── index.md                 # Phase 5: golden path overview
        ├── local-dev.md             # How to run the app locally
        └── deployment.md           # How CI/CD and Flux work together
```

---

## What the Scaffolder template does step by step

When a developer submits the "New Go App" form in Backstage:

```
Developer fills form:
  App name:    my-airline-service
  Description: Manages flight bookings
  Owner:       team-platform
         │
         ▼
Backstage Scaffolder:
  1. Creates GitHub repo: my-airline-service
  2. Renders skeleton/ files with ${{values.appName}} → my-airline-service
  3. Pushes rendered files to the new repo
  4. Registers the service in the software catalog
  5. Creates Flux HelmRelease in platform-infra for the new app
         │
         ▼
Automation takes over:
  6. GitHub Actions CI runs automatically on first push
  7. Flux deploys the app to the Kind cluster
  8. Service appears in Backstage catalog with live deployment status
```

The developer goes from zero to a running, deployed service without
knowing anything about Kubernetes, Helm, Flux, or OpenTofu.

---

## Prerequisites

- Node.js 20 LTS (installed by `00-install-tools.sh`)
- A GitHub token with `repo` and `workflow` scopes
- The Kind cluster from Phase 0 running
- The golden-app deployed (Phases 1–4 complete)

---

## Running Backstage locally (Phase 5)

```bash
# Install Node dependencies (first time only, takes ~2 min)
npm install

# Start the Backstage development server
npm run dev

# Open the portal
open http://localhost:3000
```

The first time you open the portal, you will see:
- The software catalog with `golden-app` registered
- The Scaffolder template "New Go App" ready to use
- TechDocs for the golden path

---

## Configuration overview

`app-config.yaml` is the main config file. Key sections:

```yaml
app:
  baseUrl: http://localhost:3000    # Portal URL

backend:
  baseUrl: http://localhost:7007    # Backend API URL

integrations:
  github:                           # Connects to your GitHub account
    - host: github.com
      token: ${GITHUB_TOKEN}        # Set in app-config.local.yaml

catalog:
  locations:                        # Where to find catalog-info.yaml files
    - type: file
      target: ./catalog/catalog-info.yaml

scaffolder:                         # Enables the template wizard
  defaultAuthor:
    name: Platform Team
```

Secrets (GitHub token, etc.) go in `app-config.local.yaml` which is
gitignored. Never put real credentials in `app-config.yaml`.

---

## Commit conventions

```
phase 5: backstage app-config + catalog registration
phase 5: scaffolder template for go app golden path
phase 5: techdocs — golden path developer guide
```

---

## The bigger picture

By the end of Phase 5, a new developer joining the team needs to know
exactly one thing: **open Backstage**. From there they can:

- Browse all existing services, their owners, and their docs
- See the live deployment status of every service
- Create a new production-ready service in minutes
- Find documentation without leaving the portal

This is the Internal Developer Portal pattern in practice. The platform
team maintains the templates and infrastructure. Developers consume a
self-service experience. No tickets, no waiting, no tribal knowledge.
