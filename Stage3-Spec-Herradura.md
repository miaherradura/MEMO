# Scenario 4 – U.S. Aerospace Manufacturer | FX Hedge Technical Specification

**Created by:** [Name]
**Updated by:** [Name]
**Date Created:** April 24, 2026
**Date Updated:** April 24, 2026
**Version:** 1.0
**LLM Used:** Claude Sonnet 4.6 (Anthropic)

**Role:** Financial Analyst / Treasury Analyst
**Audience:** CFO or Director of Treasury

**Purpose:** Provide a professional, quantitative specification outlining the analytical structure for evaluating FX hedging alternatives for a USD-denominated aerospace exporter with a EUR receivable.

---

## 1. Problem Statement

> Our company expects to receive EUR 20,000,000 in USD-equivalent revenue in 12 months, arising from an aerospace contract denominated in Euros. This exposure subjects us to FX risk from fluctuations in the EURUSD exchange rate. This specification outlines the analytical framework for quantifying, comparing, and evaluating alternative hedging strategies — a forward contract, a money market hedge, a EUR put option, and a EUR call option — to determine the optimal approach for protecting USD proceeds while accounting for cost and flexibility trade-offs.

Key parameters of the exposure:

- **Exposure type:** Foreign-currency receivable (EUR-denominated inflow)
- **Foreign currency amount:** EUR 20,000,000
- **Time horizon:** 12 months (1 year to maturity)
- **Objective:** Protect the USD value of the receivable against EUR depreciation while evaluating the cost of optionality
- **Decision context:** Corporate treasury function reporting to the CFO

A U.S. exporter receiving Euros is naturally **long EUR**. The primary risk is EUR depreciation against the USD, which would reduce the USD value of the receivable. The correct natural hedge is a **EUR put option** (the right to sell EUR at a fixed strike), while the forward contract locks in a guaranteed rate. This spec evaluates both, alongside a call option for reference.

---

## 2. Inputs (Known Variables)

The table below defines all model inputs. Variable names correspond directly to named cells in the accompanying Excel model. Yellow-highlighted cells in the spreadsheet denote user-editable inputs requiring current market data at the time of analysis.

| Variable | Description | Unit | Value / Example | Source |
|---|---|---|---|---|
| `FC_AMT` | Foreign-currency receivable (EUR) | EUR | 20,000,000 | Company data / scenario |
| `S₀` | Current EURUSD spot rate | USD/EUR | [Market input] | Bloomberg / Reuters |
| `F₀` | 1-year EURUSD forward rate | USD/EUR | 1.0935 | Provided in scenario |
| `r_USD` | USD 1-year risk-free interest rate | % | [Market input] | US Treasury / Fed data |
| `r_EUR` | EUR 1-year risk-free interest rate | % | [Market input] | ECB / Eurozone data |
| `t` | Time to maturity | Years | 1.0 | Derived from scenario |
| `K_put` | EUR Put strike price | USD/EUR | [Set at S₀] | Analyst choice |
| `K_call` | EUR Call strike price | USD/EUR | [Set at S₀] | Analyst choice |
| `Premium_put` | EUR Put option premium | USD/EUR | 0.019 | Provided in scenario |
| `Premium_call` | EUR Call option premium | USD/EUR | 0.024 | Provided in scenario |

> **Note:** `S₀`, `r_USD`, `r_EUR`, `K_put`, and `K_call` must be populated with current market data prior to model execution. `F₀` = 1.0935 and option premiums are fixed per scenario specification.

---

## 3. Assumptions & Constraints

The following conventions govern all calculations. Adherence ensures reproducibility by any treasury analyst or AI model builder working from this specification.

- Interest rates (`r_USD`, `r_EUR`) are quoted on a **simple annual basis** (not continuously compounded).
- The forward rate F₀ = 1.0935 represents a 1-year maturity EURUSD forward, as provided in the scenario.
- All option premiums are expressed in **USD per EUR** (no lot-size multiplier), per scenario specification.
- Option premiums are paid upfront in USD at inception; no margin or collateral treatment is modeled.
- Exchange rates are expressed as **USD per EUR** throughout (direct quote from a U.S. perspective).
- Transaction costs, bid-ask spreads, and credit risk are excluded from this model.
- No counterparty or settlement risk is modeled.
- The money market hedge assumes the company can borrow EUR at `r_EUR` and invest USD at `r_USD` without spread.
- The put and call strikes `K_put`, `K_call` are set equal to the current spot rate S₀ (at-the-money) unless overridden by the analyst.
- All proceeds are computed in USD; the reporting currency of the U.S. parent is USD.
- The model is **static (single period)**; no dynamic re-hedging or rolling strategy is evaluated.

---

## 4. Calculation Flow

The following sequence defines the order of operations for the model. Each step is self-contained and feeds the next. A junior analyst or AI model builder should be able to implement this logic in Excel without further clarification.

1. **Compute forward hedge USD proceeds.** Multiply the EUR receivable (`FC_AMT`) by the 1-year forward rate (`F₀`). This is the certainty benchmark: `USD_forward = FC_AMT × F₀`.

2. **Reconstruct a synthetic forward via money market hedge (cross-check).** (a) Borrow PV of EUR receivable at `r_EUR`: borrow `FC_AMT / (1 + r_EUR)` EUR today. (b) Convert EUR to USD at spot: `USD_proceeds = EUR_borrowed × S₀`. (c) Invest USD at `r_USD` for 1 year: `USD_mm = USD_proceeds × (1 + r_USD)`. Under Covered Interest Parity, `USD_mm` should equal `USD_forward`. Any discrepancy flags a data entry error.

3. **Build scenario table for option outcomes.** Define a range of EUR/USD spot rates at maturity S_T from 0.90 to 1.30 in steps of 0.02 (21 data points). For each S_T, compute:
   - **Unhedged:** `FC_AMT × S_T`
   - **Put hedge (net):** `FC_AMT × MAX(S_T, K_put) − FC_AMT × Premium_put`
   - **Call hedge (net, reference):** `FC_AMT × S_T − FC_AMT × Premium_call`

4. **Compute break-even spot rate for put vs. forward.** Solve for S_T where `USD_put = USD_forward`. This is the rate below which the put hedge outperforms the forward (protection kicks in) and above which the forward locks in a better net rate than the put floor.

5. **Compare all four strategies across the S_T range.** Present results in a side-by-side comparison table and a line chart showing USD proceeds vs. S_T for all strategies simultaneously.

6. **Summarize trade-offs and produce an executive recommendation.** Quantify the total put premium cost in USD, the floor proceeds guarantee, upside retained above `K_put`, and the break-even rate. Write a 1–2 paragraph conclusion suitable for CFO presentation.

---

## 5. Outputs

The table below lists all model outputs. These correspond to cells, tables, and charts in the Excel model, and serve as interpretation targets in the AI-assisted analysis stage.

| Output | Description | Format | Purpose |
|---|---|---|---|
| `USD_forward` | USD proceeds locked via forward contract | Scalar (USD) | Certainty benchmark; confirm against money market parity |
| `USD_mm` | USD proceeds via money market hedge | Scalar (USD) | Cross-check: should equal `USD_forward` under CIP |
| `USD_put` | USD proceeds using EUR put — net of premium | Table by S_T | Floor protection analysis; break-even vs. forward |
| `USD_call` | USD proceeds using EUR call — net of premium | Table by S_T | Upside / reference case; not primary hedge for exporter |
| `Chart_1` | All four strategy outcomes vs. S_T at maturity | Line chart | Visual comparison for executive presentation |
| `Break_even` | S_T at which put hedge equals forward proceeds | Scalar (rate) | Key decision threshold for option vs. forward choice |
| `Summary` | Written recommendation for CFO | 1–2 paragraphs | Executive-ready takeaway with quantified trade-offs |

> All numeric outputs should be formatted in USD with comma separators and zero decimal places (e.g., $21,870,000). Exchange rates displayed to 4 decimal places.

---

## 6. Sensitivity Plan

> Vary the EURUSD spot rate at maturity (S_T) from **0.90 to 1.30** in increments of **0.02** (21 data points). This range spans approximately ±18% around the current forward rate of 1.0935, capturing realistic stress scenarios and favorable movements. For each value of S_T, compute USD proceeds under all four strategies: Unhedged, Forward Hedge, Put Hedge (net of premium), and Call Hedge (net of premium, for reference). Present results as (1) a formatted comparison table and (2) a multi-line chart with S_T on the x-axis and USD proceeds on the y-axis.

Additional sensitivities to consider (optional extensions):

- Vary option premium by ±20% to assess sensitivity to implied volatility shifts.
- Vary tenor from 6 months to 18 months to evaluate time-value impact on option cost.
- Apply a range of interest rate differentials to test money market hedge cross-check robustness.

---

## 7. Limitations & Next Steps

### Analytical Limitations

- Implied volatility is not incorporated; option premiums are taken as given and not derived from a pricing model (e.g., Black-Scholes).
- Transaction costs, bid-ask spreads, and brokerage fees are excluded; real-world hedging costs would reduce all net proceeds.
- Credit risk and counterparty exposure on the forward contract are not modeled.
- The money market hedge assumes frictionless borrowing in EUR at the risk-free rate, which may not reflect the company's actual funding cost.
- The model is static and single-period; dynamic re-hedging, delta hedging, or rolling strategies are outside scope.
- Tax implications of hedging gains and losses are not addressed.

### Next Steps

1. Populate yellow input cells in the Excel model with current market data: `S₀`, `r_USD`, `r_EUR`, `K_put`, and `K_call`.
2. Run the money market hedge cross-check (`USD_mm` vs. `USD_forward`) to validate rate consistency under CIP.
3. Review the scenario table and line chart; identify the break-even S_T and assess probability-weighted outcomes.
4. Draft the executive summary (Stage 4) using the model outputs and this specification as the AI prompt context.
5. Present findings to the CFO or Director of Treasury with a clear recommendation: forward (certainty) vs. put option (protected upside) based on the company's risk tolerance and view on EUR direction.

---

## Appendix: How This Specification Feeds Later Stages

| Stage | What This Specification Enables |
|---|---|
| **Stage 2 – Excel Build** | Each Input variable maps to a named cell or range. Each Output becomes a computed cell or chart. |
| **Stage 3 – Model Review** | Spec documents the constructed model, articulates design choices, and flags areas for improvement. |
| **Stage 4 – AI-Assisted Analysis** | Calculation Flow becomes the AI prompt instruction block; Outputs drive interpretation and recommendation. |

> Treat this specification as the bridge between business insight and technical execution. The CFO should be confident the plan is sound even before seeing the numbers — and the model builder should be able to implement it without asking a single follow-up question.
