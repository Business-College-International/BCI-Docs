# Student Administration Milestone

## Implemented

- Added `GET /api/v1/students/directory` with server-side filters for status, level, programme, term, class and name/admission-number search.
- Privileged school staff can query the bounded directory; teachers are limited to classes covered by their teacher assignments.
- Added administrative guardian lookup at `GET /api/v1/guardians/directory`, restricted to authorized school management roles and returning only identity/contact/ward-count information needed for linking.
- Added the staff web Student Directory with filtering and server-driven scope.
- Added the staff web Student Profile panel showing core student data, the current enrolment record, linked guardians, and documents.
- Added audited guardian linking from the staff portal with explicit academic, fee and wallet permissions.
- Added controlled withdrawal from the staff portal using the existing transactional and audited backend lifecycle.
- Added backend tests for student-directory scope/filter behavior and guardian-directory authorization.

## Deliberate constraints

- The student profile currently exposes the current active enrolment, not the full historical enrolment ledger. The UI does not claim otherwise.
- Guardian unlinking remains a backend capability but is not exposed in this UI until the profile payload provides the stable guardian person identifier required for an unambiguous operation.
- Student document upload/replace remains a separate storage/security workflow; the current profile surface only displays existing document metadata and links.
- Intra-term transfer history remains blocked by the existing `@@unique([studentId, termId])` enrolment constraint until the schema redesign is completed.
