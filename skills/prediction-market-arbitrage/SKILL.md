---
name: prediction-market-arbitrage
description: Identifies mispriced outcome shares on Polymarket and evaluates arbitrage opportunities across related markets.
license: MIT
---

# Prediction Market Arbitrage

## Overview
This skill scans active binary and categorical prediction markets, identifying pricing anomalies where implied odds deviate from statistical models.

## Capabilities
- Detects mutually exclusive outcome mispricings ($P(\text{YES}) + P(\text{NO}) \ne 1.00$).
- Analyzes cross-market arbitrage opportunities across related political or financial contracts.
- Generates executable trade orders to exploit mispriced probability spreads.
