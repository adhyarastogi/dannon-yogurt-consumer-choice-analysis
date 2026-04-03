# Dannon Yogurt - Consumer Choice & Demand Analysis

**Tools:** SAS (PROC MDC, PROC SQL, PROC MEANS, PROC CONTENTS)  
**Domain:** Consumer Analytics, Discrete Choice Modeling, Pricing Strategy  

---

## Project Overview

This project models consumer purchase behavior in the Dannon yogurt category using household-level panel data across 6 SKUs observed over 52 weeks.

The goal is to quantify the role of **price**, **in-store display**, and **intrinsic brand preference** in driving consumer choice - and translate findings into actionable pricing and promotional strategy.

A **conditional logit (multinomial discrete choice) model** was implemented using `PROC MDC` in SAS to estimate:

- Own-price elasticities  
- Cross-price elasticities  
- Brand preference (via intercepts)  

Unlike standard regression, this approach models the **choice among competing alternatives**, capturing substitution patterns and brand loyalty.

---

## Dataset

Three linked SAS datasets:

| File | Observations | Variables | Role |
|------|-------------|----------|------|
| `products.sas7bdat` | 6 | 22 | Fixed product attributes |
| `prodchars.sas7bdat` | 312 | 6 | Marketing mix (price, display, feature) |
| `panels.sas7bdat` | 500 | 4 | Household purchase panel |

**Key Variables:**
- `COLUPC` (product ID)
- `IRI_KEY` (store)
- `WEEK` (time)
- `price`, `display`, `feature`
- `PANID` (household)
- `FLAVOR_SCENT`, `FAT_CONTENT`

**Choice Structure:**
- 6 products × 500 household-week decisions = **3,000 rows (long format)**

---

## Product Space

| JID | UPC | Flavor | Fat Content | Avg Price |
|-----|-----|--------|------------|----------|
| 1 | 23663200104 | Strawberry | Low Fat | $0.728 |
| 2 | 23663200108 | Blueberry | Low Fat | $0.726 |
| 3 | 23663200112 | Peach | Low Fat | $0.728 |
| 4 | 33663200620 | Peach | Nonfat Light | $0.704 |
| 5 | 33663200622 | Blueberry | Nonfat Light | $0.706 |
| 6 | 33663200625 | Strawberry | Nonfat Light | $0.690 |

All SKUs:
- 0.375 pint plastic cups  
- Non-organic  
- Differ only by flavor and fat content  

---

## Modeling Approach

### Conditional Logit Model

Utility function:

```
U_ij = α_j + β·price_ij + γ·display_ij + ε_ij
```

Where:
- `α_j` = product-specific preference
- `β` = price sensitivity
- `γ` = display effect
- `ε_ij` = IID extreme value error

Choice probability:

```
P_ij = exp(V_ij) / Σ_k exp(V_ik)
```

- 5 intercepts estimated (J1–J5)  
- JID = 6 used as reference  

---

## Key Results

### Model Comparison

| Model | Specification | Log-Likelihood | AIC | BIC | McFadden R² |
|------|--------------|---------------|-----|-----|------------|
| Model 1 | Intercepts only | −867.41 | 1745 | 1766 | 0.032 |
| Model 2 | + Price | −841.36 | 1695 | 1720 | 0.061 |
| Model 3 | + Price + Display | −839.91 | 1694 | 1723 | 0.063 |

**Selected Model:** Model 3  
- Best AIC  
- Strong economic interpretability  

---

### Parameter Estimates (Model 3)

| Parameter | Estimate | p-value | Interpretation |
|----------|---------|--------|---------------|
| Price | −6.939 | < .0001 | Strong negative price sensitivity |
| Display | +0.279 | 0.086 | Positive display effect |
| J4 (Nonfat Peach) | +0.462 | 0.001 | Most preferred SKU |
| J5 (Nonfat Blueberry) | +0.332 | 0.025 | Second most preferred |
| J1 (LF Strawberry) | −0.267 | 0.178 | Lower preference |

---

### Price Elasticity Matrix

|      | SKU1 | SKU2 | SKU3 | SKU4 | SKU5 | SKU6 |
|------|------|------|------|------|------|------|
| SKU1 | -4.012 | 0.630 | 0.519 | 0.981 | 0.840 | 0.674 |
| SKU2 | 0.381 | -3.754 | 0.519 | 0.981 | 0.840 | 0.674 |
| SKU3 | 0.381 | 0.630 | -3.885 | 0.981 | 0.840 | 0.674 |
| SKU4 | 0.381 | 0.630 | 0.519 | -3.028 | 0.840 | 0.674 |
| SKU5 | 0.381 | 0.630 | 0.519 | 0.981 | -3.196 | 0.674 |
| SKU6 | 0.381 | 0.630 | 0.519 | 0.981 | 0.840 | -3.279 |

**Insights:**
- All SKUs have **elastic demand** (|E| > 1)
- Strong substitution across products  

---

## Strategic Recommendations

### 1. Protect High-Preference SKUs
- SKU 4 (Nonfat Peach)
- SKU 5 (Nonfat Blueberry)

Actions:
- Prioritize shelf placement  
- Maintain availability  
- Invest in brand visibility  

---

### 2. Pricing Discipline
- All products are price elastic  
- SKU 1 is most sensitive  

Actions:
- Avoid price increases on high-share SKUs  
- Test increases only on low-preference items  

---

### 3. Increase Display for Weak SKUs
- SKU 1 & 2 currently have no display support  

Actions:
- Introduce 1–2 display weeks annually  
- Boost trial and awareness  

---

### 4. Manage Cannibalization
- Cross-elasticities: 0.38–0.98  

Actions:
- Avoid deep discounting low-margin SKUs  
- Protect premium products from internal competition  

---

## Analysis Workflow

1. Data Exploration → `PROC CONTENTS`, `PROC PRINT`  
2. Product ID Assignment → DATA step  
3. Descriptive Analysis → `PROC MEANS`  
4. Choice Set Assembly  
5. Many-to-Many Merge → `PROC SQL`  
6. Model 1 → Intercepts only  
7. Model 2 → + Price  
8. Model 3 → + Display  
9. Model Comparison → LogL, AIC, BIC  
10. Elasticity Computation  
11. Elasticity Matrix → `PROC TRANSPOSE`  

---

## Repository Files

| File | Description |
|------|------------|
| `Dannon_Yogurt_Consumer_Choice_Analysis.pdf` | Full analysis |
| `products.sas7bdat` | Product attributes |
| `prodchars.sas7bdat` | Marketing mix |
| `panels.sas7bdat` | Household panel data |

---

## Working with `.sas7bdat` Files

Use Python:

```python
import pyreadstat
df, meta = pyreadstat.read_sas7bdat('file.sas7bdat')
```

---

## Skills Demonstrated

- Discrete Choice Modeling  
- Conditional Logit (PROC MDC)  
- Price & Cross-Price Elasticity  
- Panel Data Analysis  
- SAS (PROC SQL, PROC MDC)  
- Consumer Behavior Analytics  
- Demand Modeling  

---
