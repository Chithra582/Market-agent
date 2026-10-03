# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Market Agent** (`market-agent`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Market Agent (`market-agent`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Finance / Autonomous Prediction Markets & Algorithmic Trading  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

The agent analyzes markets and executes prediction positions through a deterministic, 5-stage quantitative pipeline.

### 1. Decision Architecture

The runtime intake, state classification, evaluation, and execution tracking operate across a deterministic, five-stage pipeline:

```
+-----------------------------------------------------------------------------------+
|                        Deterministic Market Agent Pipeline                        |
+-----------------------------------------------------------------------------------+
|  [Stage 1: Market Universe Ingestion & Liquidity Screening]                       |
|     --> Query Polymarket CLOB/Gamma API; filter by 24h volume & bid-ask spread    |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 2: Multi-Source RAG & Evidence Synthesis]                                 |
|     --> Ingest news, polls, and betting odds via Chroma vector similarity search  |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 3: Statistical Probability Modeling & Mispricing Detection]               |
|     --> Calibrate true probability $P_{\text{est}}$ vs market implied probability $P_{\text{mkt}}$|
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 4: Risk Sizing & Portfolio Guardrail Gate]                                |
|     --> Compute fractional Kelly position size & verify maximum drawdown bounds   |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 5: Non-Custodial Order Execution & Post-Trade Auditing]                   |
|     --> Sign EIP-712 order payload, broadcast to CLOB, and record trade telemetry |
+-----------------------------------------------------------------------------------+
```

### 2. Decision Logic & Routing Formulations

Trading opportunity ranking and expected value calculation apply a probability mispricing formulation:

$$\mathbb{E}[V] = P_{\text{est}} \cdot (1 - P_{\text{mkt}}) - (1 - P_{\text{est}}) \cdot P_{\text{mkt}} = P_{\text{est}} - P_{\text{mkt}}$$

Position sizing is strictly capped using a conservative quarter-Kelly ($f^* / 4$) criterion:

$$f^* = \frac{b \cdot P_{\text{est}} - (1 - P_{\text{est}})}{b}, \quad \text{where } b = \frac{1 - P_{\text{mkt}}}{P_{\text{mkt}}}$$

$$\text{StakeSize} = \min\left(0.25 \cdot f^* \cdot \text{Equity}, \; 0.05 \cdot \text{Equity}\right)$$

Market affinity ranking across available candidate markets $m$ is evaluated by:

$$S_{\text{opp}}(m) = \alpha \cdot |\mathbb{E}[V](m)| + \beta \cdot \log_{10}(\text{Volume}_{24h}(m)) - \gamma \cdot \text{Spread}(m)$$

Where $\alpha = 0.50$, $\beta = 0.30$, and $\gamma = 0.20$.

### 3. Thresholding & Refusal Decision Criteria

Market Agent enforces strict operational boundaries and deterministic refusal thresholds:
- **Refusal on ERR_INSUFFICIENT_EDGE**: Minimum Expected Value ($\mathbb{E}[V]$) ($< 0.05$ (5% edge)) halts execution with code `ERR_INSUFFICIENT_EDGE`.
- **Refusal on ERR_EXCESSIVE_SPREAD**: Bid-Ask Spread Margin ($> 0.05$ (5 cents)) halts execution with code `ERR_EXCESSIVE_SPREAD`.
- **Refusal on ERR_LOW_MARKET_LIQUIDITY**: Minimum 24h Volume ($< \$10,000$ USD) halts execution with code `ERR_LOW_MARKET_LIQUIDITY`.
- **Refusal on ERR_DAILY_DRAWDOWN_LIMIT_REACHED**: Daily Portfolio Drawdown ($\ge 10\%$ equity drop) halts execution with code `ERR_DAILY_DRAWDOWN_LIMIT_REACHED`.
- **Refusal on ERR_INSUFFICIENT_COLLATERAL**: Private Key / Balance Check (Insufficient USDC) halts execution with code `ERR_INSUFFICIENT_COLLATERAL`.

### 4. Fallback Decision Mechanism

Continuous operational stability is maintained through layered fault recovery:
- **Tier 1 (Automated Order Cancel & Retry)**: If a limit order remains unfilled for 120 seconds while market prices move, the order is automatically canceled and repriced.
- **Tier 2 (Defensive Derisking)**: If contradictory highimpact breaking news emerges, the agent automatically executes position hedge or partial unwinding.
- **Model Fallback Cascade**: High-level reasoning and synthesis default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

Human operators retain sovereign authority over the multi-agent execution lifecycle:
- **Tier 3 (Human Supervision Escalation)**: Orders exceeding \$2,500 USD or trades involving contested resolution criteria trigger a mobile webhook prompt requiring human manual approval.
- **Benchmark Trajectory Auditing**: Operators inspect evaluation traces, raw generation tokens, and container logs to verify scoring fidelity.

---

## The Data It Uses

Market Agent operates under strict principles of data minimization, environment isolation, and privacy protection.

### 1. Ingested Input Data

The framework processes only operational data necessary to perform its functions:
- **Polymarket Market Data**: CLOB order books, trade ticks, outcome share prices (YES/NO), and resolution conditions.
- **External Real-World Signals**: News feeds (NewsAPI, Perplexity), polling data (FiveThirtyEight, Silver Bulletin), and social sentiment.
- **Wallet Telemetry**: Polygon USDC balances, active token allowances, and open position exposures.

### 2. Configuration & Reference Data

- **Chroma Vector Database**: Embedding store indexing past event resolutions and news context.
- **EVM Blockchain Node**: Polygon mainnet RPC endpoints for transaction settlement verification.

### 3. Base Model & Inference Lineage

- **Inference Engines**: OpenAI GPT-4o, Claude 3.5 Sonnet via LangChain reasoning pipelines.
- **Execution Stack**: `py-clob-client`, Web3.py, Python 3.9+, Docker.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against indirect prompt injection, credential leakage, and unauthorized external API dispatch.
- **Local Environment Isolation**: Agent execution workspaces, intermediate scratchpads, and vector stores reside strictly within designated local project directories.
- **Automated Secret Scrubbing**: API keys, database credentials, and personal credentials are automatically redacted prior to embedding or logging.
- **Zero Commercial Monetization**: Prompts, intermediate reasoning trajectories, and task deliverables are never commercialized or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of Market Agent is essential for effective deployment.

### 1. Oracle Resolution Ambiguity & Delays
- **Limitation**: Polymarket market resolutions rely on UMA optimistic oracle votes, which may undergo disputes or prolonged delays.
- **Mitigation**: Disallow trading in markets with ambiguous or poorly drafted resolution criteria.

### 2. Latency Asymmetry Against High-Frequency Bots
- **Limitation**: High-frequency algorithmic market makers co-located near CLOB matching engines possess sub-millisecond speed advantages.
- **Mitigation**: Focus on medium-to-long horizon fundamental forecasting rather than sub-second latency arbitrage.

### 3. Flash Illiquidity Spikes Around Breaking News
- **Limitation**: Sudden geopolitical or election news can cause market makers to pull quotes, leaving wide empty order books.
- **Mitigation**: Mandate strict post-only limit orders to avoid paying exorbitant market order taker fees during liquidity voids.

### 4. Model Hallucination in Event Probability Sizing
- **Limitation**: LLM reasoning may exhibit overconfidence or hallucinate dates regarding political timelines.
- **Mitigation**: Ground all probability estimates in numerical polling data and verified news citations via RAG verification.

### 5. Smart Contract & Blockchain RPC Downtime
- **Limitation**: Polygon network congestion or RPC node rate limits can delay order submission and cancellation.
- **Mitigation**: Configure multi-endpoint fallback RPC providers (Alchemy, Infura, QuickNode) with automatic retry handlers.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & routing formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested input data & query streams | Section 1 | Verified |
| - Configuration & reference schemas | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Oracle Resolution Ambiguity & Delays | Section 1 | Verified |
| - Latency Asymmetry Against High-Frequency Bots | Section 2 | Verified |
| - Flash Illiquidity Spikes Around Breaking News | Section 3 | Verified |
| - Model Hallucination in Event Probability Sizing | Section 4 | Verified |
| - Smart Contract & Blockchain RPC Downtime | Section 5 | Verified |
