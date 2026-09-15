# Public Website Milestone

## Completed

- Reworked `bci-website` from a single application form into a public school website.
- Added sticky navigation and responsive mobile navigation treatment.
- Added About, education pathway, SHS programmes, Admissions, application tracking and Contact sections.
- Application form now distinguishes SHS programme selection from KG/JHS submissions.
- Added public application status lookup against the authoritative `/applications/track/:trackingCode` API.
- Added success/error handling and retained application tracking codes after submission.
- Added SEO/social metadata, canonical metadata and theme color.
- Added `robots.txt` and `sitemap.xml`.
- Added PR/manual-only website build verification.

## Deliberate boundaries

- The public site does not expose authenticated student/staff operations.
- No school address, phone number, fees, programme requirements, or other facts were invented where authoritative BCI data was not present.
- Official school content such as exact contact details, term dates, prospectus, elective combinations and admission requirements should be added from approved school records later.
