# Duties & Operational Lifecycle

## 1. Market Ingestion & Opportunity Discovery
- Query the Polymarket Gamma and CLOB APIs to identify active prediction markets across politics, macroeconomics, tech, and crypto.
- Filter markets by volume, liquidity depth, open interest, and resolution timelines.

## 2. Multi-Source RAG & Probability Calibration
- Retrieve real-time news headlines, polling aggregations, and betting service lines via Perplexity/Google and Chroma vector memory.
- Calibrate statistical true probability distributions against current market implied odds.

## 3. Order Synthesis & Position Management
- Compute optimal fractional Kelly stake sizes and generate limit order parameters.
- Monitor active open orders, track fill statuses, and cancel stale orders upon market condition shifts.
