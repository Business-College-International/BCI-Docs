# BCI Messaging and Timetable Schema Gate

## Messaging / staff chat

The current PostgreSQL schema has announcements and notification deliveries but no durable conversation model. A production chat feature should be introduced as a deliberate migration with at least:

- `Conversation` — direct/group/thread identity, type, title, createdBy, archivedAt.
- `ConversationParticipant` — user membership, role, joinedAt, leftAt, notification preference, lastReadMessageId.
- `Message` — conversationId, senderUserId, body, messageType, createdAt, editedAt, deletedAt, replyToMessageId.
- `MessageAttachment` — messageId, object/file reference, media type, size and checksum.
- `MessageRead` is optional if participant last-read pointers cannot provide required auditing.

Required authorization rules:

- Users can read/write only conversations in which they are active participants.
- Directors/admins may have moderation capabilities, but must not gain silent access to private conversations merely from role ownership.
- Group conversations require an explicit participant-management action.
- Deleted/edited messages retain audit metadata.
- Attachments must use controlled object storage and content-type/size validation.
- Push/SMS/email notifications are delivery channels, not the source of truth for messages.
- Message delivery retries must be idempotent.

## Timetable

The current schema contains teacher assignments, classes, subjects and terms but no timetable entity. A production timetable should therefore be introduced as versioned scheduling data rather than a mutable list of teacher assignments.

Required structure:

- `TimetableVersion` — academicYearId, termId, status (`DRAFT`, `PUBLISHED`, `RETIRED`), versionNumber, createdBy, publishedAt.
- `TimetableEntry` — versionId, classId, subjectId, teacherAssignmentId, dayOfWeek, periodId, roomId.
- `TimetablePeriod` — school-relative period code/name, start/end time and ordering.
- `TimetableRoom` — room identity/capacity where room scheduling is needed.

Publication should be immutable. A new published timetable creates a new version; existing published entries are never edited in place.

Validation must prevent conflicts for:

- teacher + day + period
- class + day + period
- room + day + period

The published timetable should be the source for teacher and class views in web/mobile. Attendance sessions may reference timetable context, but attendance records remain authoritative independently.

No timetable or conversation tables should be added until the Prisma migration contract is executed and verified against a real PostgreSQL instance.
