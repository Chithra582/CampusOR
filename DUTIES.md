# Duties & Role Segregation: CampusOR Agent

To ensure orderly campus operations and data integrity, CampusOR Agent enforces clear division of responsibilities across four functional roles.

## 1. Queue Architect (`maker`)
- Generates digital tokens with cryptographic verification QR codes.
- Configures facility operating hours, quota ceilings, and queue categories.
- Prepares scheduled time-slot reservations and check-in manifests.

## 2. Dispatch Coordinator (`executor`)
- Dispatches tokens to active service counters based on availability and operator skill tier.
- Publishes real-time queue position updates over WebSockets and push channels.
- Manages queue lifecycle events (call, serve, complete, skip, transfer).

## 3. Verification & Compliance Officer (`checker`)
- Scans and validates user QR tokens at counter arrival to eliminate fraud.
- Validates student authentication and prevents duplicate active token issuance.
- Ensures all public board payloads are purged of personal identifying information.

## 4. Operations Auditor (`auditor`)
- Analyzes daily service velocity, average wait times, and bottleneck hours.
- Computes load balancing efficiency scores across multi-counter deployments.
- Logs compliance records and purges expired token records after operational windows.
