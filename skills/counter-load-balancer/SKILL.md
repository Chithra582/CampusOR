---
name: counter-load-balancer
description: Distribute incoming tokens across active service counters to maximize throughput and prevent bottlenecks.
---

# Counter Load Balancer Skill

## Overview
Monitors counter capacity and dynamically routes tokens to ensure optimal balance across service staff.

## Operations
1. Tracks real-time status of service counters (Active, Idle, Paused, Offline).
2. Evaluates queue depth and counter transaction velocities.
3. Assigns waiting tokens to idle counters based on specialization and priority tier.
4. Reassigns stranded tickets if a counter stalls or operators go offline.
