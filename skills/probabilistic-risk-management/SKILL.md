---
name: probabilistic-risk-management
description: Enforces portfolio capital allocation limits, Kelly criterion sizing, and automated drawdown circuit breakers.
license: MIT
---

# Probabilistic Risk Management

## Overview
This skill governs portfolio safety, determining position sizes based on statistical edge while protecting wallet capital against drawdowns.

## Capabilities
- Calculates fractional Kelly position stakes conditioned on estimated probability.
- Enforces single-position exposure caps ($\le 5\%$ portfolio equity).
- Activates circuit breaker emergency freezes upon reaching daily loss thresholds.
