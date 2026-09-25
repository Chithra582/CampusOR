# Explainability & Transparency Report: CampusOR Agent

> **Specification:** OpenGAP v0.1.0  
> **Domain:** Productivity / Campus Queue Management & Operations  
> **Target System:** CampusOR (Campus Online Queue & Reservation System)  
> **Audit Status:** Qualified for HiDevs GitAgent Passport  

---

## 1. Overview & Operational Purpose

**CampusOR Agent** is an autonomous queue management and predictive dispatch agent designed for **CampusOR**. The agent transforms crowded physical waiting areas into streamlined virtual queue environments across universities, hospital clinics, administrative registrar offices, and cafeterias.

Core agent responsibilities include:
1. **Virtual Token Sequencing**: Automated ticket generation, QR validation, and state machine progression.
2. **Multi-Counter Dispatch**: Real-time workload distribution among active counters to minimize bottlenecks.
3. **ML Wait-Time Estimation**: Dynamic duration calculation combining historical completion times, real-time counter count, and queue volume.
4. **Multi-Channel Notification Dispatch**: Automated dispatch of arrival alerts via WebSockets, SMS, WhatsApp, and Telegram.

---

## 2. How the Agent Decides (Decision-Making Logic)

```
User Request (Join Queue / Book Slot / Operator Next / Check Status)
  │
  ├── 1. Intake Validation & Rate Limiting
  │      ├── Verify user authorization (JWT / Student ID)
  │      ├── Check facility operating hours and capacity ceiling
  │      └── Issue sequential token + unique QR cryptographic payload
  │
  ├── 2. Machine Learning Wait-Time Calculation
  │      ├── Gather metrics: queue_length (L), active_counters (C), avg_service_time (S)
  │      ├── Predict Estimated Wait Time (EWT) = (L / C) * S * peak_hour_coefficient
  │      └── Broadcast initial countdown to user client
  │
  ├── 3. Dynamic Multi-Counter Dispatching
  │      ├── Monitor counter states (Idle, In-Service, Paused)
  │      ├── When Counter reports IDLE:
  │      │   ├── Match next eligible token by category and priority
  │      │   ├── Assign token to Counter ID
  │      │   └── Trigger "You're Being Called" priority notification
  │      └── If operator marks "Skip / No-Show":
  │          └── Apply 3-minute grace period before terminal archival
  │
  └── 4. Kiosk & Display Privacy Sanitization
         ├── Redact user name and private reason
         └── Stream token code (e.g., "B-201 → Counter 3") to public displays
```

---

## 3. Data Sources & Inputs Used

| Data Input | Source | Purpose | Data Handling & Privacy |
|---|---|---|---|
| **User Identity & Token Request** | Student / Visitor Web & Mobile App | Generating unique token and scheduling queue entry | Encrypted with JWT; PII redacted from all public displays |
| **Counter State & Velocity** | Operator Staff Console | Tracking active window capacity, status, and service times | Operational metadata; logged for load analytics and auditing |
| **Historical Service Records** | MongoDB Service History Database | Training ML models for accurate wait-time estimation | Anonymized completion durations; purged of personal data |
| **Channel Preferences** | User settings (WhatsApp, WebPush, SMS) | Routing "You're Next" proximity notifications | Phone numbers and endpoints encrypted; zero third-party marketing |

---

## 4. Known Limitations & Failure Modes

### 1. Inaccurate Wait Times from Outlier Transactions
*Limitation:* Complex student inquiries or irregular medical consultations can cause sudden delays that disrupt linear wait-time models.  
*Mitigation:* The agent continuously re-estimates wait times using moving averages with outlier clipping, immediately sending proactive push notifications to waiting users when unexpected delays occur.

### 2. User No-Shows and Zombie Tokens
*Limitation:* Users who abandon the virtual queue without cancellation inflate perceived queue length and cause operators to call empty seats.  
*Mitigation:* The agent sends proximity pings at 3-position intervals, requiring users to confirm presence within a 3-minute grace window; unconfirmed tokens are automatically routed to a temporary recall queue before expiration.

### 3. Network Disconnects & Offline Kiosks
*Limitation:* Intermittent campus Wi-Fi drops can cause client apps or display kiosks to lose real-time WebSocket state.  
*Mitigation:* The system implements progressive web app (PWA) offline caching with exponential backoff WebSocket reconnection and automated state reconciliation upon re-link.

### 4. Counter Imbalance from Staff Unavailability
*Limitation:* Sudden operator departures without pausing the counter cause assigned tokens to stall indefinitely.  
*Mitigation:* The agent detects counter inactivity exceeding 1.5x average service time, automatically flagging the stall, unbinding stranded tokens, and re-dispatching them to alternate open counters.

---

## 5. Verification, Safety & Human Oversight

1. **Role-Based Access Control (RBAC)**: Strict segregation between student/visitor clients, counter operators, and system administrators.
2. **Audit Trail Logging**: Every token transition (issued, called, served, skipped, cancelled) is recorded in immutable structured JSON logs.
3. **Human Staff Override**: Counter operators maintain full sovereignty to manually override agent queue ordering, call specific urgent cases, or pause queues at will.
4. **FERPA & Privacy Protection**: Public displays and broadcast channels reveal only alphanumeric token numbers, strictly eliminating student names and academic or clinical identifiers.
