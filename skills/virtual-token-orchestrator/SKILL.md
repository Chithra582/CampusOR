---
name: virtual-token-orchestrator
description: Generate digital queue tokens, validate QR check-ins, and manage token lifecycle states.
---

# Virtual Token Orchestrator Skill

## Overview
Governs the creation, verification, and state transitions of virtual queue tokens across all campus departments.

## Operations
1. Issues sequential alphanumeric tokens with embedded cryptographic verification QR codes.
2. Validates user check-ins at facility entrances and kiosks.
3. Manages state machine transitions: `ISSUED` → `WAITING` → `CALLED` → `SERVING` → `COMPLETED` / `SKIPPED`.
4. Enforces slot reservations and prioritizes scheduled appointments.
