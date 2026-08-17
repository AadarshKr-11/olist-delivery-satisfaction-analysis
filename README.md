# Delivery Delay vs Customer Satisfaction — Olist E-Commerce

**Late deliveries make up 6.8% of orders but generate 36.6% of all 1-star reviews — over five times their share.** The penalty is not proportional to how late an order is: missing the promised date by a single day costs 1.2 stars, and being a month late costs barely more than being two weeks late. Delivering *early*, meanwhile, earns almost nothing. The lever is hitting the promised date, not shortening it.

---

## The question

Olist is a Brazilian e-commerce marketplace. Review scores are its most visible customer-satisfaction signal, and delivery is the part of the experience the platform most directly controls.

I wanted to answer one question with an actionable answer: **how much does late delivery actually cost in customer satisfaction, and where should Olist intervene?**

## Data

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle). Four tables used:

| Table | Rows | Used for |
|---|---|---|
| `orders` | 99,441 | Order status, purchase and delivery timestamps |
| `order_reviews` | 99,224 | Review scores (1–5) |
| `order_items` | 112,650 | Product-level order composition |
| `products` | 32,951 | Product categories |

Analysis is on the **96,478 delivered orders**, of which 96,361 had a review.

## Method

1. **Type conversion.** All five date columns loaded as text and were converted to datetime.
2. **Scoped to delivered orders.** ~3,000 orders had no delivery date; these matched non-delivered statuses (cancelled, unavailable, shipped, processing), so the nulls represent meaningful absence rather than missing data. Delivery delay is undefined for an order that never arrived.
3. **Computed delay** as `actual delivery date − estimated delivery date`. Positive = late, negative = early.
4. **Joined reviews** at order level (inner join, one review per order — verified no fan-out).
5. **Bucketed the delay** to test whether the satisfaction penalty is gradual or sharp.
6. **Tested category variation** by joining order items and products.

## Findings

### 1. Olist delivers 12 days early on average

Mean delay is **−11.9 days**; median **−12**. The 75th percentile is −7, so at least three-quarters of orders arrive ahead of the estimate. Only **6.8% of orders are actually late**.

This is not exceptional logistics — it is substantial padding built into the delivery estimate.

### 2. Late orders score two full stars lower

| | Average review score |
|---|---|
| On time or early | **4.29** |
| Late | **2.27** |

### 3. The penalty is a cliff, not a slope

| Delay bucket | Avg review score | Change |
|---|---|---|
| More than 10 days early | 4.32 | — |
| 0–10 days early | 4.22 | −0.10 |
| **1–5 days late** | **2.99** | **−1.23** |
| **6–15 days late** | **1.75** | **−1.24** |
| More than 15 days late | 1.73 | −0.02 |

Three things follow:

- **Being early buys almost nothing.** Ten extra days of earliness is worth 0.1 stars.
- **The damage happens at day one.** Customers punish the *fact* of a broken promise, not its magnitude.
- **It floors out by two weeks.** Once an order is badly late, being later costs nothing more.

### 4. Lateness is cross-cutting, not category-specific

I expected poorly-rated categories to be running later. They are not — average delay sits between −11.2 and −12.5 days across **every** category with meaningful volume.

| Category | Avg review score | Avg delay (days) | Items |
|---|---|---|---|
| moveis_escritorio | 3.52 | −11.8 | 1,664 |
| cama_mesa_banho | 3.92 | −11.7 | 10,985 |
| moveis_decoracao | 3.95 | −12.5 | 8,159 |
| casa_construcao | 3.96 | −11.5 | 593 |
| informatica_acessorios | 3.98 | −12.5 | 7,672 |

Office furniture is the worst-rated category but sits mid-pack on delay. Whatever drives its low scores — product quality, damage in transit for bulky items, expectation mismatch — **this dataset cannot say**, and I have not speculated.

The useful conclusion is the negative one: **delivery is a platform-wide problem, not a category problem.** The fix is not category-specific logistics.

### 5. 6.8% of orders cause 36.6% of 1-star reviews

If lateness had no effect, late orders would produce roughly their fair share of 1-star reviews — about 6.8% of them. They produce **36.6%**: over five times their share.

## Recommendation

**Compete on reliability, not speed.**

1. **Treat the promised date as the metric that matters.** Early delivery generates no measurable satisfaction gain; missing the date destroys it. Internal targets should be built around estimate accuracy, not average transit time.

2. **Consider tightening estimates — carefully.** Twelve days of padding produces no goodwill and makes Olist look slower than it is at the point of purchase, where the estimate influences conversion. Any tightening must preserve the on-time rate, since the downside is severe and immediate.

3. **Escalate at the moment a promise is at risk, not after.** Because the entire penalty is incurred on day one of lateness, intervention has to happen *before* the estimated date passes. Once an order is a day late the damage is done and further delay is nearly free — so recovery effort is better spent preventing the first day than rescuing week three.

4. **Investigate low-rated categories separately.** Office furniture and bed/bath underperform for reasons unrelated to delivery. That requires product-level data this analysis does not include.

## Caveats

- **Correlation, not causation.** Late delivery may co-occur with other failures (stock issues, seller problems) that independently affect satisfaction. This analysis establishes association and magnitude, not mechanism.
- **2,965 orders were never delivered** and are excluded. Their customers may be the least satisfied of all, so the true cost of delivery failure is likely *understated* here.
- **117 delivered orders had no review** and were dropped by the inner join — 0.1% of the sample, no material effect.
- **Category analysis is item-level.** An order containing five items appears five times, so the metric is properly read as *"average review score of orders containing at least one item from this category"* — a low score may be caused by a different item in the same order. I chose this over forcing one category per order, which would have fabricated a fact the data does not contain.
- **`ROWS`-style bucketing.** Delay buckets are fixed ranges chosen for interpretability, not derived from the distribution.
- Review scores are self-selected; customers with strong feelings are more likely to leave one.

## Reproducing

```
delivery_delay_impact_analysis.ipynb
```

Requires `pandas`. Download the four CSVs from the Kaggle link above and update the `path` variable at the top of the notebook.

---

**Aadarsh Kumar** · [LinkedIn](https://www.linkedin.com/in/adz11/) · adz.aadarsh.11@gmail.com
