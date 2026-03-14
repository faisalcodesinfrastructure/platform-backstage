# platform-backstage

Backstage Internal Developer Portal for the Platform Engineering Golden Path workshop.

Backstage is the developer-facing front door. Developers use it to discover
services, create new apps from templates, and read documentation — without
needing to know anything about Kubernetes, Helm, or Flux.

---

## Structure (built in Phase 5)

```
platform-backstage/
├── app-config.yaml             Phase 5 — main Backstage config
├── app-config.local.yaml       Local secrets — never committed
├── catalog/
│   └── catalog-info.yaml       Phase 5 — service catalog entry
├── templates/
│   └── go-app-template/
│       ├── template.yaml       Phase 5 — Scaffolder form + steps
│       └── skeleton/           Phase 5 — files copied to new repos
└── techdocs/docs/              Phase 5 — Markdown documentation
```

---

## Running locally (Phase 5)

```bash
npm install
npm run dev
open http://localhost:3000
```

---

## Phase instructions

Phase 5 instructions are added to this README in Phase 5.
Complete Phases 0–4 before starting Phase 5.
