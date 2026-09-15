# BCI Docs — Business College International School Management System

Shared source of truth for the BCI system: architecture, data model, API
contracts, and design decisions. All other repos link back here.

## Start here
**[MASTER_PLAN.md](./MASTER_PLAN.md)** — full architecture, roles, modules,
tech stack, core data model, security notes (Ghana Data Protection Act 843),
and the phased build roadmap (Phase 0 → 7).

## Repositories (GitHub org: [Business-College-International](https://github.com/Business-College-International))

| Repo | Purpose | Stack |
|---|---|---|
| [bci-backend-api](https://github.com/Business-College-International/bci-backend-api) | The core — single source of truth for identity, students, staff, academics, finance, wallet, stationery, payroll, announcements, messaging | Node.js + TypeScript + NestJS + Prisma + PostgreSQL |
| [bci-web-portal](https://github.com/Business-College-International/bci-web-portal) | Internal portal for office staff, accountants, principal, director | Vite + React + TypeScript |
| [bci-website](https://github.com/Business-College-International/bci-website) | Public marketing site + online application entry point | Vite + React + TypeScript |
| [bci-mobile-app](https://github.com/Business-College-International/bci-mobile-app) | Guardians/parents + staff app (fees, wallet, attendance, timetable, payslips) | Flutter (Dart) |
| bci-shared-contracts *(planned, once API stabilizes)* | OpenAPI spec + shared types generated from the backend | — |

## Directory layout
- `MASTER_PLAN.md` — the master architecture & build plan
- `decisions/` — architecture decision records (one file per decision)
- `erd/` — entity-relationship diagrams (mermaid / dbdiagram.io exports)

## Ground rules
1. All business logic lives in `bci-backend-api` — clients stay thin. Never let the web portal and mobile app develop divergent logic.
2. Role checks are enforced server-side on every endpoint.
3. One identity system (RBAC over a single `users` table) — never separate login systems per user type.
4. Build in roadmap order (Phase 0 → 7 in MASTER_PLAN.md section 9), not all at once.
