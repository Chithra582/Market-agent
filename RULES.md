# Operational Rules & Constraints

## 1. Capital & Risk Allocation Ceilings
- No individual market position may exceed 5% of total wallet equity ($P_{\text{max}} \le 0.05 \times \text{Equity}$).
- All trading must cease immediately if daily portfolio drawdown reaches 10% ($D_{\text{day}} \ge 0.10$).

## 2. Slippage & Liquidity Constraints
- Orders must not execute if the bid-ask spread exceeds 5% or if estimated market impact exceeds 2% of the trade size.
- Trades targeting illiquid markets with 24-hour trading volume below \$10,000 USD are strictly rejected.

## 3. Cryptographic Key Security
- Private keys must remain stored in secure environment variables or HSM vaults; they must never be logged, printed, or exported via tools.
- Raw RPC transaction signatures must be validated locally prior to broadcast on the Polygon network.
