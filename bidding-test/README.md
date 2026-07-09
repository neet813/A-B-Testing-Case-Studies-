# Average Bidding vs Maximum Bidding — Don't Trust the First Metric

## The situation

A company runs online ad campaigns and wants to know whether switching from their current bidding strategy (**maximum bidding**, the test group) is better than their old one (**average bidding**, the control group). The dataset has 80 days of campaign data — 40 days on each strategy — with impressions, clicks, purchases, and earnings tracked daily.

The obvious first question: does the new strategy lead to more purchases?

## Step 1: Check purchases

```python
control = df[df['Group'] == 'control']['Purchase']
test = df[df['Group'] == 'test']['Purchase']
stat, p_value = stats.ttest_ind(control, test)
```

Result: **p = 0.35**. Not significant. Purchase counts are basically the same between the two strategies — control averages 550.9 purchases a day, test averages 582.05. That gap could easily just be random noise.

On its own, this looks like a "do nothing, not enough evidence" result.

## Step 2: But purchase count isn't the actual business question

Purchases tell you *how many* people bought — not how much money came in. The dataset also has an `Earning` column, which is the real business metric here. I hadn't checked it yet, so I did:

```python
control_earning = df[df['Group'] == 'control']['Earning']
test_earning = df[df['Group'] == 'test']['Earning']
stat_e, p_value_e = stats.ttest_ind(control_earning, test_earning)
```

Result: **p ≈ 0.0000**. Hugely significant.

- Control average earning: 1,908.58
- Test average earning: 2,514.93
- That's a **31.8% lift** in revenue for the test group (maximum bidding).

## Step 3: What this actually means

The two strategies produce roughly the same *number* of purchases, but the purchases under maximum bidding are worth a lot more money. That makes sense once you think about what "maximum bidding" does — it's not aiming to win more auctions, it's aiming to win the *right, higher-value* ones.

If I'd stopped after checking Purchase count, I would have told the business "no clear winner, wait for more data" — which would have been the wrong call. The real answer was sitting in a column I hadn't looked at yet.

## Final recommendation

**Switch to maximum bidding.** It doesn't increase how many people buy, but it significantly increases how much revenue each campaign generates — a 31.8% lift, with very strong statistical confidence (p≈0.0000). Purchase count was the wrong metric to judge this decision on; earnings is the one that matters.

## The takeaway

Always check whether the metric you're testing is actually the metric the business cares about. A "no difference" result on the wrong metric can hide a very real, very significant difference on the right one.

