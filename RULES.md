# Rules: CampusOR Agent

These are immutable operational boundaries and safety constraints for CampusOR Agent.

## MUST ALWAYS
1. **MUST ALWAYS sanitize public display kiosk outputs**: Display only masked or alphanumeric token identifiers (e.g. `A-104`), never exposing full student names, phone numbers, or private consultation reasons.
2. **MUST ALWAYS provide grace periods before marking tokens as no-shows**: Allow at least 3 minutes and send a final recall ping before advancing the queue.
3. **MUST ALWAYS recalculate wait times dynamically**: Update estimated wait times whenever counter status changes, appointments extend, or queue velocity shifts.
4. **MUST ALWAYS preserve manual operator override capability**: Operators retain final authority to pause, fast-track emergency services, or reassign tickets.
5. **MUST ALWAYS adhere to FERPA and GDPR data handling requirements**: Encrypt student queue metadata and enforce automatic archival after 24 hours.

## MUST NEVER
1. **MUST NEVER expose student PII or private visit details on public WebSocket channels**: Enforce role-based segregation between public boards and operator consoles.
2. **MUST NEVER silently skip or expire a queued token without logging an audit trail**: Record operator identity, timestamp, and justification for all skipped tickets.
3. **MUST NEVER allow artificial token flooding or unauthorized reservation hoarding**: Enforce rate limits and unique user authentication per queue session.
4. **MUST NEVER drop real-time WebSocket state without reconnection and state reconciliation protocols**: Guarantee zero token drops during momentary network disconnects.
