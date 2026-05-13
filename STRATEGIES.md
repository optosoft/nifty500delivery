# Trading Strategies - Detailed Guide

This document explains how each strategy works with real example calculations.

## Base Filter (Applied to All Strategies)

Before any strategy filtering, all stocks must meet the **base delivery criteria**:

```
deliveryEod > deliveryAvg6Month  AND
deliveryAvgWeek >= minDeliveryPct (default 60%)  AND
deliveryAvgMonth >= minDeliveryPct (default 60%)
```

This ensures we're only looking at stocks with sustained high delivery (institutional interest).

---

## 1. Momentum Strategy 🚀

**Logic:** Trend-following strategy that scores stocks showing upward momentum across multiple indicators.

### Scoring Rules (1 point each):
1. Current Price > 200-day SMA
2. Current Price > 50-day SMA
3. 50-day SMA > 200-day SMA (trend is up)
4. Price Change % > 0% (positive today)
5. Price Change % > 2% (strong positive move)

**Minimum Threshold:** Score ≥ 4 (passes 4 out of 5 signals)

### Example Calculation

Stock: RELIANCE

| Signal | Check | Value | Pass? |
|--------|-------|-------|-------|
| Price > SMA200 | 2850 > 2700 | ✓ | 1 point |
| Price > SMA50 | 2850 > 2800 | ✓ | 1 point |
| SMA50 > SMA200 | 2800 > 2700 | ✓ | 1 point |
| Price Change > 0% | +1.5% > 0% | ✓ | 1 point |
| Price Change > 2% | +1.5% > 2% | ✗ | 0 points |

**Momentum Score: 4/5** ✓ **PASSES (≥4)**

---

## 2. Reversion Strategy ⚡

**Logic:** Mean reversion strategy that identifies stocks that have pulled back despite strong institutional accumulation.

### Scoring Rules (1 point each):
1. Current Price < 200-day SMA
2. Current Price < 50-day SMA
3. Price Change % > 0% (bouncing despite being below average)
4. Delivery EOD > Delivery 6-month Average (fresh institutional buying)

**Minimum Threshold:** Score ≥ 2 (passes 2 out of 4 signals)

### Example Calculation

Stock: INFY

| Signal | Check | Value | Pass? |
|--------|-------|-------|-------|
| Price < SMA200 | 1650 < 1750 | ✓ | 1 point |
| Price < SMA50 | 1650 < 1700 | ✓ | 1 point |
| Price Change > 0% | +0.8% > 0% | ✓ | 1 point |
| Del EOD > Del 6M Avg | 65% > 58% | ✓ | 1 point |

**Reversion Score: 4/4** ✓ **PASSES (≥2)**

---

## 3. Volume Strategy 📊

**Logic:** Identifies stocks experiencing unusual delivery concentration, suggesting institutional accumulation.

### Scoring Rules (1 point each):
1. Delivery EOD > Delivery Week Average (unusual activity this week)
2. Delivery EOD > Delivery Month Average (unusual activity this month)
3. Delivery EOD > 70% (very high delivery, institutional focus)
4. Price Change % > 0% (volume supporting upside)

**Minimum Threshold:** Score ≥ 3 (passes 3 out of 4 signals)

### Example Calculation

Stock: TCS

| Signal | Check | Value | Pass? |
|--------|-------|-------|-------|
| Del EOD > Del Week Avg | 72% > 68% | ✓ | 1 point |
| Del EOD > Del Month Avg | 72% > 65% | ✓ | 1 point |
| Del EOD > 70% | 72% > 70% | ✓ | 1 point |
| Price Change > 0% | -0.5% > 0% | ✗ | 0 points |

**Volume Score: 3/4** ✓ **PASSES (≥3)**

---

## 4. Composite Strategy ⭐

**Logic:** Blended approach combining momentum, reversion, and volume signals with custom weights.

### Scoring Formula:

```
Composite Score = (Momentum × 0.40) + (Reversion × 0.40) + (Volume × 0.20)
```

**Weights:**
- Momentum: 40% - Uptrend following
- Reversion: 40% - Bounce potential
- Volume: 20% - Accumulation confirmation

**Minimum Threshold:** Score ≥ 3.0

### Example Calculation

Stock: HDFC BANK (using scores from above examples)

| Component | Score | Weight | Contribution |
|-----------|-------|--------|---------------|
| Momentum | 4 | 0.40 | 1.60 |
| Reversion | 4 | 0.40 | 1.60 |
| Volume | 3 | 0.20 | 0.60 |

**Composite Score: 1.60 + 1.60 + 0.60 = 3.80** ✓ **PASSES (≥3.0)**

---

## Real-World Scenario Analysis

### Scenario 1: Strong Momentum Play

**Stock:** BHARTI AIRTEL

Base Filter Status:
- Delivery EOD: 75% > 60% Min ✓
- Delivery 6M Avg: 68% ✓
- Week/Month Avg: 62% ✓

Strategy Scores:
- **Momentum: 5/5** (Price above both SMAs, bullish trend, +2.3% change)
- **Reversion: 1/4** (Price above SMAs, doesn't qualify)
- **Volume: 4/4** (All delivery signals firing)
- **Composite: (5×0.40) + (1×0.40) + (4×0.20) = 3.6**

**Strategy Results:**
- ✓ Passes Momentum (5 ≥ 4)
- ✗ Fails Reversion (1 < 2)
- ✓ Passes Volume (4 ≥ 3)
- ✓ Passes Composite (3.6 ≥ 3.0)

**Best for:** Momentum traders seeking uptrend continuation

---

### Scenario 2: Mean Reversion Opportunity

**Stock:** MARUTI SUZUKI

Base Filter Status:
- Delivery EOD: 68% > 60% Min ✓
- Delivery 6M Avg: 55% ✓
- Week/Month Avg: 61% ✓

Strategy Scores:
- **Momentum: 1/5** (Price below both SMAs, -1.2% change)
- **Reversion: 3/4** (Both prices below SMAs, bouncing, fresh accumulation)
- **Volume: 2/4** (Moderate delivery)
- **Composite: (1×0.40) + (3×0.40) + (2×0.20) = 2.0**

**Strategy Results:**
- ✗ Fails Momentum (1 < 4)
- ✓ Passes Reversion (3 ≥ 2)
- ✗ Fails Volume (2 < 3)
- ✗ Fails Composite (2.0 < 3.0)

**Best for:** Contrarian traders betting on bounce-back

---

### Scenario 3: Balanced Opportunity

**Stock:** INFOSYS

Base Filter Status:
- Delivery EOD: 70% > 60% Min ✓
- Delivery 6M Avg: 62% ✓
- Week/Month Avg: 65% ✓

Strategy Scores:
- **Momentum: 4/5** (Price above 50-day SMA, trend intact, +0.8% change, SMA50 > SMA200)
- **Reversion: 2/4** (Price between SMAs, delivery accelerating)
- **Volume: 3/4** (All delivery signals except price weakness)
- **Composite: (4×0.40) + (2×0.40) + (3×0.20) = 3.0**

**Strategy Results:**
- ✓ Passes Momentum (4 ≥ 4)
- ✓ Passes Reversion (2 ≥ 2)
- ✓ Passes Volume (3 ≥ 3)
- ✓ Passes Composite (3.0 ≥ 3.0) - Barely!

**Best for:** All-around investors seeking balanced signals

---

## Tips for Strategy Selection

### Use Momentum When:
- Market is in uptrend
- You see consistent higher highs and higher lows
- SMAs are properly aligned (50 > 200)
- Multiple timeframe confirmations are present

### Use Reversion When:
- Stock is extended down despite institutional support
- Delivery is increasing while price is down (accumulation)
- You expect mean reversion to SMAs
- Institutional accumulation on weakness signals value

### Use Volume When:
- Delivery is unusually concentrated
- Week/Month delivery averages spike
- You want to follow smart money
- Institutional accumulation patterns are visible

### Use Composite When:
- You want a balanced view
- You're uncertain about market direction
- You want to filter out extreme plays
- You prefer diversified signal confirmation

---

## Key Parameters You Can Adjust

### Delivery Threshold (Default: 60%)
- **Lower (40-50%):** Captures more stocks, higher noise
- **Higher (70-80%):** Stricter filter, only concentrated delivery plays

### Strategy Thresholds
These are hardcoded but represent optimal balance:
- Momentum minimum: 4/5 (80% signal confirmation)
- Reversion minimum: 2/4 (50% signal confirmation)
- Volume minimum: 3/4 (75% signal confirmation)
- Composite minimum: 3.0/5.0 (60% weighted score)

---

## Common Questions

**Q: Can a stock pass multiple strategies?**  
A: Yes! A stock can pass Momentum + Volume but fail Reversion, for example. Use any combination.

**Q: Why is Composite weighted 40-40-20?**  
A: Momentum and Reversion are equally important (opposite views of market structure). Volume is confirmation weight.

**Q: What's the relationship between SMA50 and SMA200?**  
A: SMA50 = short-term trend, SMA200 = long-term trend. SMA50 > SMA200 = bullish alignment.

**Q: Does "Delivery %" mean institutional ownership?**  
A: Delivery % measures shares held past settlement (low turnover). High delivery + rising delivery = institutional accumulation pattern.

**Q: Should I use all strategies together?**  
A: No. Each tells a different story. Use one or combine strategically:
- Momentum + Volume = Institutional-led uptrend
- Reversion + Volume = Smart money accumulating weakness
- All three = Maximum confirmation (rare)
