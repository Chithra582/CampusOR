# CampusOR  
**Campus Online Queue & Reservation System**

[![OpenGAP Spec 0.1.0](https://img.shields.io/badge/OpenGAP-0.1.0-blue.svg)](https://opengitagent.org)
[![GitAgent Passport](https://img.shields.io/badge/GitAgent%20Passport-Ready-brightgreen.svg)](https://app.hidevs.xyz/passport/submit)
[![Category](https://img.shields.io/badge/Category-Productivity-teal.svg)](https://app.hidevs.xyz/passport/submit)
[![Compliance](https://img.shields.io/badge/Compliance-FERPA%20%7C%20GDPR-orange.svg)](EXPLAINABILITY.md)

CampusOR is a smart, scalable virtual queue and reservation system for large campuses such as universities, hospitals, hostels, cafeterias, and administrative offices. It replaces physical queues with a real-time digital experience, reducing congestion and improving service efficiency.

---

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Objectives](#objectives)
- [Key Features](#key-features)
- [System Architecture](#system-architecture)
- [Technology Stack](#technology-stack)
- [Folder Structure](#folder-structure)
- [Novelty and Uniqueness](#novelty-and-uniqueness)
- [Expected Outcomes](#expected-outcomes)
- [Future Enhancements](#future-enhancements)
- [Conclusion](#conclusion)

---

## Overview

Managing queues in large campuses often leads to long wait times, overcrowding, and inefficiencies. CampusOR introduces a virtual queue ecosystem where users can join queues digitally, track their position in real time, and receive timely notifications—without standing in physical lines.

The system combines real-time communication, predictive analytics, and role-based dashboards to deliver a seamless queue management experience.

---

## Problem Statement

Traditional campus queue systems face:

- Long physical queues and overcrowding  
- No real-time queue visibility  
- Manual token handling and errors  
- Inefficient multi-counter coordination  
- No waiting-time prediction  
- Fragmented systems across campus  

---

## Objectives

CampusOR aims to:

- Digitize queue and token management  
- Reduce physical waiting and crowding  
- Enable real-time tracking and alerts  
- Support multi-counter operations  
- Predict waiting times using data  
- Provide a unified campus-wide solution  

---

## Key Features

### Users
- Join queues digitally via web or PWA  
- Real-time queue position tracking  
- Smart notifications (“You’re Next”)  
- QR-based token verification  
- Time-slot booking and rescheduling  
- Offline PWA support with auto-sync  

### Operators
- Accept, skip, or recall tokens  
- Pause and resume queues  
- Multi-counter coordination  
- Kiosk mode for public displays  

### Admin
- Queue and counter management  
- Load balancing across counters  
- Analytics: peak hours, wait time, efficiency  
- Role-based access control  

### Machine Learning
- Predicts waiting time using:
  - Historical service data  
  - Queue length  
  - Counter availability  
  - Time-based patterns  

---

## System Architecture

- Clients communicate via REST APIs and WebSockets  
- Backend manages queues, tokens, and users  
- Real-time server syncs live updates  
- ML service predicts wait times  
- Databases store queue and analytics data  
- Notification services push real-time alerts  

---

## Technology Stack

- **Frontend:**  Next.js 
- **Backend:** Node.js, Express  
- **Database:** MongoDB   
- **Real-Time:** WebSockets, Socket.IO  
- **ML:** Python (FastAPI)  
- **Notifications:** WhatsApp, Telegram, Email (which ever is easier to integrate) 
- **Deployment:** Docker, Vercel, AWS  
- **Security:** JWT, RBAC  

---

## Folder Structure

```txt
campusor/
├── frontend/        # Next.js 
├── backend/         # Express + MongoDB API
├── ml-service/      # FastAPI ML service
├── shared/          # shared types & schemas
├── infra/           # Docker & deployment
└── docs/            # architecture & API docs
```

---

## GitAgent Passport Qualification

This repository is fully compliant with the **OpenGAP Spec 0.1.0** standard and qualified for the **HiDevs GitAgent Passport**:

- **Checkpoint 1 (Validate):** Verified OpenGAP spec 0.1.0 compliance via [`agent.yaml`](agent.yaml), [`SOUL.md`](SOUL.md), [`skills/`](skills/), and [`tools/`](tools/).
- **Checkpoint 2 (Explain):** Comprehensive 5-section transparency report in [`EXPLAINABILITY.md`](EXPLAINABILITY.md) detailing virtual queue dispatch logic, ML wait-time regression models, FERPA/GDPR compliance, and failure mode mitigations.
- **Checkpoint 3 (Export):** Cross-framework export compatibility tested across OpenAI SDK, CrewAI, Claude Code, and Lyzr.
- **Target Category:** **`Productivity`** (Campus Queue Management & Operations).

