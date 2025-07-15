# Aave V2 DeFi Wallet Credit Scoring

This project builds a credit scoring model for Ethereum wallets using historical transaction data from the **Aave V2 protocol**. Each wallet is assigned a score between **0 and 1000**, where a higher score indicates responsible usage (e.g. regular repayments, no liquidations), and a lower score suggests risky or bot-like behavior.

---

## Challenge Statement

> You are given 100K raw, transaction-level records from Aave V2. Each record corresponds to actions such as `deposit`, `borrow`, `repay`, `redeemunderlying`, or `liquidationcall` by a wallet.  
> Your task is to develop a robust model that assigns a credit score (0–1000) to each wallet based solely on its historical behavior.

---

## Solution Overview

The entire pipeline is implemented in two notebooks:

### 1. **Feature Engineering**
File: `Aave_Credit_Scoring_Part1_Data_Processing.ipynb`

- Loaded and flattened the raw JSON file
- Extracted wallet-level features by grouping actions:
  - Total transactions (`txn_count`)
  - Number of deposits, borrows, repays
  - Liquidation count
  - Total deposited, borrowed, repaid in USD

---

### 2. **Credit Score Generation**
File: `Aave_Credit_Scoring_Part2_Scoring_and_Analysis.ipynb`

- Normalized key features using `MinMaxScaler`
- Computed a custom weighted score:
  - Rewards high deposits and timely repayments
  - Penalizes liquidations and excessive borrowing
- Mapped final score to a **0–1000** scale
- Plotted score distribution histogram
- Saved output to `wallet_scores.csv`

---

## Features Used for Scoring

| Feature | Description |
|---------|-------------|
| `txn_count` | Total protocol actions per wallet |
| `num_deposits` | Number of deposits made |
| `num_borrows` | Number of borrow actions |
| `num_repays` | Number of repay actions |
| `num_liquidations` | How often the wallet was liquidated |
| `total_deposited_usd` | Total USD value deposited |
| `total_borrowed_usd` | Total USD value borrowed |
| `total_repaid_usd` | Total USD value repaid |

---

## Scoring Logic

We created a scoring function combining the above features using this formula:

```python
score = (
    0.2 * total_repaid_usd_scaled +
    0.2 * num_repays_scaled +
    0.15 * total_deposited_usd_scaled +
    0.15 * txn_count_scaled +
    0.10 * num_deposits_scaled +
    0.10 * (1 - total_borrowed_usd_scaled) +
    0.10 * (1 - was_liquidated)
)
