# Business College International (BCI) — School Management System
## Master Architecture & Build Plan

**School profile assumed for this plan:** KG → JHS → SHS, public-private (GES-contracted) school in Ghana, SHS offering Agric, General Arts, Business, and Home Economics, each with electives and class divisions (e.g. General Arts A/B/C).

---

## 1. Executive Summary

This is four connected products sharing one backend and one database:

1. **Backend API** — Node.js, the single source of truth for all data and business logic.
2. **Mobile App (Flutter)** — for parents/guardians, students (where relevant), and staff/teachers.
3. **Web Portal (Vite + React)** — for office staff, principal, director, accounts/finance, and admissions.
4. **Public Website (Vite + React)** — marketing + online application entry point for the general public.

One PostgreSQL database. One set of roles and permissions. One notification/payment layer (Moolre). Everything else is a "view" onto that core.

Given the real complexity here (finance, payroll, attendance, admissions, messaging, e-commerce, wallets), the biggest risk isn't the tech — it's scope. Section 12 gives you a phased build order so you ship something usable in weeks, not launch everything at once in year two.

---

## 2. Roles & Access Model

Design this as **one identity system, many role profiles** — not separate login systems per user type.

| Role | Access | Primary surface |
|---|---|---|
| **Director** | Full system access, all financial flow, all staff records, payroll approval, expenditure/disbursement approval, cross-branch reporting | Web portal + mobile (read-heavy dashboards) |
| **Principal** | Academic oversight, staff duty assignment, attendance oversight, discipline records — below Director, above HOD/teachers | Web portal + mobile |
| **Office/Admin staff (Bursar/Registrar clerks)** | Data entry: admissions, student records, fee collection, receipts, stationery orders, wallet top-up/withdrawal processing | Web portal (primary tool) |
| **Accountant/Finance staff** | Expense entry, payment reconciliation, payroll processing, daily cash flow entry | Web portal |
| **Teacher** | Timetable, assigned classes/subjects, attendance marking, gradebook (future), messaging, own salary/payslip view | Mobile app (+ optional web) |
| **Non-teaching staff (cleaners, security, etc.)** | Duty roster, messaging, own salary/payslip view | Mobile app |
| **Guardian/Parent** | Applicant registration, ward's profile, bills & payment, receipts, wallet (pocket money), attendance view, stationery store, announcements/chat with school | Mobile app (primary), can also apply via website |
| **Applicant (not yet admitted)** | Application form + status tracking only | Mobile app or website |

**Key design decision:** one `users` table with role-based access control (RBAC), not separate systems. A guardian account can be linked to multiple wards (siblings). A staff account can be both a teacher AND hold an admin duty (common in smaller schools) — so permissions should be **assigned by role(s) plus specific permission grants**, not hardcoded per job title.

---

## 3. Core Modules

### 3.1 Identity, Applications & Registration
- Single sign-up flow: register as Guardian, Staff, or start as Applicant.
- **Online application**: KG/JHS/SHS applicant form — bio-data, passport photo upload, previous school, guardian info, course/programme preference (SHS only) with electives.
- Admissions review workflow in the web portal (pending → under review → admitted/rejected → enrolled), with SMS notification to guardian at each stage.
- On admission: applicant record converts into a full **Student** record, assigned to a class/division.

### 3.2 Student Information System (SIS)
- Bio-data: full name, DOB, hometown, region, passport photo, previous school attended, guardian(s) name/phone/relationship/occupation, home address, emergency contact, medical notes.
- Academic: level (KG1–SHS3), programme (Agric/General Arts/Business/Home Econ for SHS), class + division (e.g. "General Arts B"), electives, academic year/term, status (active/graduated/withdrawn/transferred).
- Document storage: passport photo, birth certificate, previous results/transcript (file uploads).
- One student can have multiple guardians linked (mother, father, guardian) with a "primary contact" flag for SMS.

### 3.3 Staff Management
- Staff profile: bio-data, role(s), employment date, contract type, department, qualifications.
- **Duty assignment**: what a staff member is responsible for (admin duty, exam duty, etc.), visible on their own dashboard.
- **Teacher-specific**: subjects taught, classes assigned, timetable (day/period/subject/class/room), attendance-taking permission scoped to their classes.
- **Timetable engine**: build once per term in web portal, published to all affected teachers' mobile dashboards automatically.

### 3.4 Attendance
- Teachers mark attendance per class/period from the mobile app (or web).
- Guardian sees their ward's attendance in near-real-time.
- Principal/Director see aggregate attendance dashboards (by class, by student, by date range) — flags chronic absenteeism.
- Staff attendance/clock-in (optional Phase 2) feeds into payroll deductions if the school wants that.

### 3.5 Finance — Fees, Payments, Receipts
- **Bill generation**: office defines fee structures per level/programme/term (tuition, feeding, PTA levy, etc.), auto-applied to enrolled students, itemized.
- **Payment collection**: guardian pays in-app via **Moolre** (Mobile Money collection). Partial payments supported; running balance shown.
- **Receipts**: auto-generated digital receipt (PDF) on every successful payment, stored in guardian's "Payment History" and accessible offline.
- **Reconciliation**: office/accounts dashboard shows daily collections, outstanding balances by class/student, exportable reports.

### 3.6 Guardian Wallet ("Pocket Money" account)
- Guardian tops up a ring-fenced wallet per student via Moolre.
- Student withdraws physical cash at the school office in increments; office staff deduct from the digital wallet at time of withdrawal (staff-initiated debit, requires office login + student verification).
- Full ledger per student: top-ups, withdrawals, running balance — visible to guardian and office.
- This is **not** the same balance as fee payments — keep these as two clearly separate ledgers to avoid confusion (fees vs. spending money).

### 3.7 Stationery Store
- Simple catalogue (exercise books, uniform items, etc.) managed by office staff in the web portal.
- Guardian orders and pays via Moolre (or draws from the wallet above — decide which; recommend **separate payment**, not wallet, to keep the pocket-money ledger simple).
- Office fulfills order; item handed to student at school; order status tracked (ordered → ready → collected).

### 3.8 Payroll & Expenditure (Director/Finance view)
- Salary structure per staff member (base pay + allowances - deductions), pay cycle (e.g. monthly).
- Finance staff process payroll runs; **Moolre disbursement API** pays staff via mobile money (or marks as bank-paid manually).
- Staff mobile app shows: payslip history, payment status (paid/pending).
- **Director dashboard**: real-time cash flow — collections in, expenses out, payroll committed, net position — daily/weekly/termly views. Also: who's been paid this cycle vs outstanding.
- **Expense entry**: office/accounts staff log expenditures with category, amount, receipt/attachment, approver. Director can see full expenditure trail.

### 3.9 Communication
- **In-app staff chat/messaging**: direct messages + role/department group channels (e.g. "SHS Teachers", "Office Staff"). Simple, not a full Slack clone — Phase 2 candidate if timeline is tight.
- **Announcements → SMS**: office/director/principal broadcasts an announcement; system fires SMS via Moolre's messaging API to all relevant guardians (by class, by level, or whole school), and shows in-app too.
- **Payment/disbursement SMS**: auto-SMS on successful fee payment, wallet top-up, wallet withdrawal, and salary disbursement — this is largely transactional and can piggyback on Moolre's collection/disbursement webhooks rather than being a separate manual step.

### 3.10 Website (public)
- School info, programmes offered (with the 4 SHS courses + electives), admissions info, contact, news/announcements.
- "Apply Now" button deep-links into the same application flow as the mobile app (same backend endpoint).

---

## 4. Recommended Tech Stack

You already have strong instincts here — this confirms and fills in the gaps:

| Layer | Choice | Notes |
|---|---|---|
| Mobile app | **Flutter (Dart)** | Single codebase for guardian/staff Android+iOS. Most Ghanaian parents are on Android — prioritize that first. |
| Backend | **Node.js + TypeScript**, framework **NestJS** | Plain Express works, but NestJS gives you modular structure (auth module, students module, finance module, etc.) which matches how complex this system actually is — you'll thank yourself in month 3. |
| Database | **PostgreSQL** | Correct choice — relational integrity matters a lot here (fees ↔ students ↔ guardians ↔ payments must never drift). |
| ORM | **Prisma** | Type-safe, great migrations, pairs well with TS/NestJS. |
| Web portal (staff/office) | **Vite + React + TypeScript**, **TanStack Query** for data fetching, **shadcn/ui** or MUI for components | |
| Public website | **Vite + React** (or even simpler: Astro, if you want faster static pages + better SEO) | Keep this separate from the internal portal — different audience, different auth (mostly none). |
| Auth | **JWT access + refresh tokens**, backend-issued, role claims embedded | Same auth service used by mobile, web portal, and website's applicant flow. |
| File storage | S3-compatible object storage (AWS S3, or Cloudflare R2 to save cost) | Passport photos, receipts, documents. |
| SMS + Payments | **Moolre API** — collections (fee/wallet/stationery payment), disbursements (payroll/refunds), SMS | One provider covers three of your requirements — good. |
| Push notifications | Firebase Cloud Messaging (Flutter has first-class support) | For in-app chat + announcement delivery on mobile. |
| Hosting | Backend: Render/Railway/Fly.io to start (cheap, simple) → migrate to AWS/DigitalOcean as you scale. DB: managed Postgres (same provider or Neon/Supabase). | Don't over-invest in infra before you have users. |
| CI/CD | GitHub Actions | Lint/test/build on PR, auto-deploy on merge to main. |

---

## 5. Data Model (Core Entities)

This is not exhaustive but gives you the backbone to start Prisma schema design:

```
User (id, email/phone, password_hash, status, created_at)
  └─ UserRole (user_id, role)  [director|principal|office|accountant|teacher|support_staff|guardian|applicant]

Guardian (user_id, occupation, address)
  └─ GuardianStudent (guardian_id, student_id, relationship, is_primary_contact)

Staff (user_id, staff_id_no, department, employment_date, contract_type, salary_id)
  └─ StaffDuty (staff_id, duty_description, assigned_by, active)
  └─ TeacherAssignment (staff_id, class_id, subject_id, term_id)
  └─ Timetable (class_id, subject_id, teacher_id, day_of_week, period, room)
  └─ SalaryStructure (staff_id, base_pay, allowances[], deductions[])
  └─ PayrollRun (staff_id, period, gross, net, status, moolre_txn_ref, paid_at)

Student (id, first_name, last_name, dob, hometown, region, passport_photo_url,
         previous_school, level, programme, class_id, division, status, admitted_at)
  └─ StudentDocument (student_id, type, file_url)
  └─ Elective (student_id, subject_id) [SHS only]
  └─ Attendance (student_id, class_id, date, status, marked_by)

Application (id, applicant_bio_data..., level_applied, programme_applied,
             status[pending|review|admitted|rejected], reviewed_by, submitted_at)

Class (id, name, level, programme, division, academic_year)
Subject (id, name, level, is_elective, programme)

FeeStructure (id, level, programme, term, item_name, amount)
StudentBill (id, student_id, fee_structure_id, term, amount_due, amount_paid, status)
Payment (id, student_id, guardian_id, amount, purpose[fee|wallet_topup|stationery],
         moolre_txn_ref, status, receipt_url, created_at)

Wallet (student_id, balance)
WalletTransaction (wallet_id, type[topup|withdrawal], amount, processed_by, created_at)

StationeryItem (id, name, price, stock_qty)
StationeryOrder (id, student_id, guardian_id, items[], total, status, moolre_txn_ref)

Expense (id, category, amount, description, receipt_url, entered_by, approved_by, created_at)

Announcement (id, title, body, audience_scope, sent_by, sms_sent, created_at)
Message (id, sender_id, recipient_id_or_channel, body, sent_at, read_at)
```

---

## 6. System Architecture (high level)

```
                         ┌─────────────────────┐
                         │   PostgreSQL (RDS)   │
                         └──────────▲───────────┘
                                    │ Prisma
                         ┌──────────┴───────────┐
                         │   Backend API (Nest)  │
                         │  REST, JWT auth, RBAC │
                         └───┬─────────┬─────────┘
              ┌──────────────┤         ├───────────────┐
              │              │         │               │
      ┌───────▼──────┐ ┌────▼────┐ ┌──▼───────┐ ┌──────▼──────┐
      │  Moolre API   │ │   S3    │ │   FCM    │ │  (future)   │
      │ SMS/Pay/Disb  │ │ Storage │ │  Push    │ │  Email svc  │
      └───────────────┘ └─────────┘ └──────────┘ └─────────────┘
                                    │
      ┌─────────────────────────────┼─────────────────────────────┐
      │                             │                              │
┌─────▼─────┐              ┌────────▼────────┐            ┌────────▼────────┐
│  Flutter   │              │   Web Portal    │            │  Public Website │
│ Mobile App │              │ (Vite + React)  │            │ (Vite + React)  │
│ guardians/ │              │ office/finance/ │            │  info + apply   │
│ staff      │              │ principal/dir.  │            │                 │
└────────────┘              └─────────────────┘            └─────────────────┘
```

All three client apps talk to **one** backend API. Never let the web portal and mobile app develop divergent business logic — logic (fee calculation, RBAC, payroll math) lives server-side only; clients are thin.

---

## 7. Repository Structure

**Recommendation: separate repos under one GitHub organization**, not a single monorepo — Flutter and the JS/TS apps have very different tooling, and separate repos give you cleaner CI pipelines and cleaner access control (e.g. you may eventually want to grant a contract developer access to just the mobile repo).

Suggested GitHub org: `bci-school-system` (or your school's actual short name), with repos:

1. **`bci-backend-api`** — NestJS + Prisma + PostgreSQL. The core.
2. **`bci-web-portal`** — Vite + React staff/office/director portal.
3. **`bci-website`** — Vite + React public site.
4. **`bci-mobile-app`** — Flutter app (guardians + staff).
5. **`bci-docs`** — this plan, ERD, API contracts, design decisions — the shared source of truth so all repos stay in sync.
6. *(optional, once the API stabilizes)* **`bci-shared-contracts`** — OpenAPI spec / shared TypeScript types generated from the backend, consumed by web-portal and website.

I can't create GitHub repos directly on your account from here (no GitHub access token), but I've scaffolded the exact folder structure and starter files for all of these locally — see the attached zip. Steps to go live:
```bash
# for each repo:
cd bci-backend-api && git init && git remote add origin git@github.com:<org>/bci-backend-api.git
git add . && git commit -m "chore: initial scaffold" && git push -u origin main
```

---

## 8. Security & Compliance Notes (Ghana context)

- Handle this under **Ghana's Data Protection Act, 2012 (Act 843)** — you're processing children's personal data (DOB, photos, home address) and financial data. Register as a data controller with the Data Protection Commission if you haven't already; this is a real legal requirement, not optional polish.
- Passwords: hashed (bcrypt/argon2), never stored/logged in plain text.
- Payment data: never store raw Mobile Money PINs or card numbers — Moolre handles the payment collection UI/tokenization, your backend only stores transaction references and status.
- Guardian ↔ Student linkage must be verifiable at registration (e.g. staff-verified at enrollment) to prevent unauthorized people from viewing a child's data.
- Role checks enforced **server-side** on every endpoint — never trust the client to hide a button and call that "access control."
- Audit log on sensitive actions: wallet withdrawals, payroll runs, expense approvals, admission status changes — who did what, when.

---

## 9. Phased Build Roadmap

Building everything simultaneously will stall the project. Build in this order — each phase is independently useful:

**Phase 0 — Foundations (2–3 weeks)**
Repo scaffolds, DB schema + migrations, auth (register/login/roles), CI/CD pipelines, hosting set up.

**Phase 1 — Core SIS + Admissions (3–4 weeks)**
Application flow (public website + mobile), admissions review (web portal), student records, class/programme/division setup, guardian-student linking. *This alone is already usable for the next admissions cycle.*

**Phase 2 — Finance Core (3–4 weeks)**
Fee structures, bill generation, Moolre payment collection, digital receipts, office collections dashboard.

**Phase 3 — Staff & Academics (3–4 weeks)**
Staff profiles, duty assignment, timetable, teacher mobile dashboard, attendance marking + guardian attendance view.

**Phase 4 — Payroll & Director Dashboard (2–3 weeks)**
Salary structures, payroll runs, Moolre disbursement, director cash-flow dashboard, expense entry.

**Phase 5 — Wallet, Stationery Store, Announcements/SMS (3 weeks)**
Pocket-money wallet + office withdrawal flow, stationery catalogue + orders, announcement broadcast + Moolre SMS.

**Phase 6 — Messaging/Chat (2 weeks)**
Staff-to-staff messaging, channels. Deliberately last — nice-to-have, not core operations.

**Phase 7 — Polish & Scale**
Reporting/exports, performance tuning, push notifications, offline support in Flutter for spotty connectivity, staff clock-in if wanted.

Realistic estimate for Phases 0–5 (the operationally essential system) with a small, focused team: **4–6 months**. Phase 6–7 can trail afterward.

---

## 10. What I've scaffolded for you right now

In the attached zip:
- Folder structure for all 4 apps + docs, ready to become 4 (or 5) separate git repos.
- `docker-compose.yml` for local PostgreSQL.
- Starter Prisma schema reflecting section 5's core entities.
- README in each app folder explaining its purpose and first setup steps.
- `.gitignore`s appropriate to each stack.

## 11. Immediate next steps for you
1. Decide: Flutter targeting Android-first or Android+iOS from day one (affects testing effort).
2. Get a **Moolre merchant account** set up now — sandbox/test credentials — this always takes longer than expected to provision.
3. Confirm exact fee structure line items and current SHS elective combinations per programme with the school office — needed before Phase 2.
4. Push the scaffolded repos to GitHub under an org, and I can help you build out Phase 0 (auth + schema) next.
