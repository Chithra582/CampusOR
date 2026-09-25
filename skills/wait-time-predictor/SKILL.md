---
name: wait-time-predictor
description: Forecast user wait times using queue volume, counter availability, and historical service durations.
---

# Wait Time Predictor Skill

## Overview
Computes dynamic estimated wait times (EWT) and target service windows for queued participants using machine learning algorithms.

## Operations
1. Ingests current queue length, active counter capacity, and time-of-day traffic parameters.
2. Combines moving-average transaction times with historical regression models.
3. Updates user arrival ETAs continuously as queue velocity shifts.
4. Detects long-tail outlier cases and informs downstream notification services.
