# Account Security Milestone

## Completed

- Authenticated users can change passwords with current-password verification.
- New passwords require at least 12 characters plus upper, lower, number and special character constraints through the DTO validation boundary.
- Password changes rotate `tokenVersion`, revoke all refresh sessions, and create an audit record.
- Authenticated users can list their own refresh sessions without any token material.
- Authenticated users can revoke an individual session only when it belongs to their own account.
- Authenticated users can revoke all sessions.
- Revoking all sessions also increments `tokenVersion`, so already-issued access tokens fail the JWT strategy immediately.
- Web staff and guardian portals expose the same security workspace.
- Flutter staff and guardian applications expose password and session controls.

## Security boundaries

- Refresh-token values and password hashes are never returned to clients.
- Session ownership is enforced by `userId` on the server.
- Guardian and staff clients share the same authenticated security endpoints; role-specific school permissions remain separate from account security.
- Password changes require the existing password and cannot reuse it.

## Deliberate limitations

- Sessions currently expose timestamps/status only; device/browser metadata is not inferred because the current schema does not store it.
- Password reset via email/SMS or recovery codes remains a separate recovery workflow and has not been guessed into the current implementation.
- Multi-factor authentication remains a future security-design gate.
