# BCI Domain Model

## Modeling principles

The school domain needs history. Current assignment and historical fact must not be the same record. Prefer effective-dated relationships and enrolment records over overwriting fields.

## Organization and academic calendar

```text
School
 ├─ AcademicYear
 │   └─ Term
 └─ SchoolLevel (KG/JHS/SHS)
      ├─ Programme
      ├─ Subject
      ├─ Class/Stream
      └─ TimetablePeriod
```

A class is an offering for a specific academic year/term context. A programme such as BUSINESS is configuration, not an application-code branch.

## People and identity

```text
User ──< UserRole >── Role ──< RolePermission >── Permission
  │
  └── Person
        ├── GuardianProfile ──< GuardianStudent >── Student
        ├── StaffProfile
        └── ApplicantProfile
```

A single person may have multiple operational relationships. Authentication credentials live on User; domain identity lives on Person/profile records.

## Admissions and enrolment

```text
Applicant/Application
       │ admission decision
       ▼
Student
       │
       └──< StudentEnrolment >── AcademicYear/Term/Class/Programme
```

Admissions are historical records. Enrolment is the bridge between a student and an academic context. Promotion creates a new enrolment context.

## Student records

Student contains stable identity/bio-data. Supporting records include documents, contacts, medical/other sensitive information, disciplinary records if adopted, and enrolment history.

The student record should not embed a single permanent `classId` that destroys history.

## Academics

```text
TeacherAssignment ── Staff + Subject + Class + Term
TimetableEntry ───── TeacherAssignment + Period + Room
AttendanceSession ── Class + Subject/Period + date
AttendanceRecord ─── Student + Session + result
Assessment ───────── Subject + Class/Term
AssessmentResult ─── Student + Assessment + score/grade
```

Attendance and assessment records are append-oriented and attributable to the actor/time of capture.

## Finance

```text
FeePlan → FeeCharge/Invoice → Payment → PaymentAllocation
                                      └── Receipt

Payment/Refund/Expense/Payroll/Wallet → Journal/FinancialLedger
```

Do not store fee balance as the authoritative accounting truth. It is derived from charges minus valid allocations/adjustments.

## Wallet

```text
StudentWallet
 └─ WalletTransaction
      ├─ TOP_UP
      ├─ WITHDRAWAL
      └─ ADJUSTMENT/REVERSAL
```

Wallets are per-student and guardian-controlled for top-up. Physical withdrawal is an office workflow with operator attribution.

## Inventory

```text
Product → StockMovement
Order → OrderLine → Product
Order → Payment
Order → Fulfilment
```

Stock on hand is derived from movements or maintained transactionally with movement history. Orders reference price snapshots so historical receipts remain accurate after catalogue price changes.

## Staff and payroll

```text
StaffProfile
 ├─ Employment/Position
 ├─ DutyAssignment
 ├─ TeacherAssignment
 ├─ CompensationStructure
 └─ PayrollEntry ── PayPeriod
                    └─ DisbursementAttempt
```

Salary changes should be effective-dated. A payslip records the calculation used for that pay period; later salary edits must not rewrite prior payroll history.

## Communications

```text
Announcement → AudienceResolution → NotificationDelivery
Conversation → ConversationMember → Message
```

Audience membership is resolved by backend rules. Delivery attempts are separate durable records for retry and audit.

## Audit and integration

All privileged mutations produce audit records. External provider interactions create provider transaction/event records so ambiguous states can be reconciled without guessing.
