
# Spanish Translation A/B Test — Catching a Misleading Result

## The situation

An e-commerce site sells across Spain and Latin America. For years, every Spanish-speaking country used the same translation, written by someone from Spain. A data scientist noticed Spain converted way better than every other country, and the team guessed it might be the translation — so they ran a test: every country except Spain got its own local translation (Argentinian for Argentina, Mexican for Mexico, and so on). Spain didn't change at all, since there was nothing to test there.

When the results came in, the test looked **negative** — the new local translations were converting *worse* than the old one. That's surprising. Why would a Mexican user prefer a translation written by someone in Spain over one written by a Mexican?

## Step 1: Is the negative result even real?

Yes. I ran a z-test on conversion rate, control vs test:

- Control: 5.52%
- Test: 4.34%
- z = 18.19, p ≈ 0

With over 450,000 users, this gap is not random noise. Something real is going on.

## Step 2: But is it really the translation's fault?

Here's the catch. Spain converts at 7.97% — way above everyone else, who mostly sit around 5%. And because Spain never got a new translation, **every single Spain user is sitting only in the control group.** The test group has no equivalent high-converting country to balance that out.

So the two groups aren't just seeing different translations — they're made up of different people. Control is being pulled up by Spain's presence. This is a classic case of **Simpson's Paradox**: the aggregate result can flip once you account for a confounding subgroup.

I excluded Spain and re-ran the test:

- Control (no Spain): 4.83%
- Test: 4.34%
- z = 7.38, p ≈ 0

Still significant. So Spain explains *some* of it — but not all of it. Something else is broken.

## Step 3: Finding the second problem

Next I checked whether every country actually got a fair, even split between test and control. Most did — ratios sitting right around 1.0. But two countries didn't:

- **Argentina**: test group had ~4x more users than control
- **Uruguay**: test group had ~9x more users than control

That's not a translation effect. That's **broken randomization** — a Sample Ratio Mismatch (SRM). Something went wrong in how these two countries got assigned to test vs control.

## Step 4: The real answer

I excluded Spain, Argentina, and Uruguay together and ran the test one more time:

- Control: 5.01%
- Test: 5.04%
- z = -0.36, p = 0.72

No significant difference. Gone.

**The localised translations never actually hurt conversion.** The entire "negative" result was caused by two unrelated problems stacked on top of each other — Spain's one-sided presence in control, and broken randomization in Argentina and Uruguay. Once both are accounted for, there's no evidence the new translations were worse.

## Step 5: Making sure this doesn't slip through again

A significant p-value only tells you two groups are different — not that the comparison was fair. So I wrote a small function that checks any future test for both of these failure modes automatically:

1. **Population imbalance** — does any subgroup exist in only one arm of the test?
2. **Sample Ratio Mismatch** — does any subgroup's test:control split ratio look far off from the overall ratio?

Running it on this dataset immediately flags all three problem countries — Spain, Argentina, and Uruguay — without me having to manually dig for them.

```python
def check_experiment_validity(df, group_col, test_col, srm_ratio_threshold=1.5):
    issues = []
    overall_counts = df[test_col].value_counts()
    overall_ratio = overall_counts.get(1, 0) / max(overall_counts.get(0, 1), 1)

    group_counts = df.groupby(group_col)[test_col].value_counts().unstack(fill_value=0)
    group_counts.columns = ['control_n', 'test_n']

    one_arm_only = group_counts[(group_counts['control_n'] == 0) | (group_counts['test_n'] == 0)]
    if not one_arm_only.empty:
        issues.append(f"Population imbalance: {list(one_arm_only.index)} appear in only one arm.")

    group_counts['ratio'] = group_counts['test_n'] / group_counts['control_n'].replace(0, pd.NA)
    srm_flags = group_counts[
        (group_counts['ratio'] > overall_ratio * srm_ratio_threshold) |
        (group_counts['ratio'] < overall_ratio / srm_ratio_threshold)
    ]
    if not srm_flags.empty:
        issues.append(f"Sample Ratio Mismatch: {list(srm_flags.index)} have a skewed split.")

    return {"valid": len(issues) == 0, "issues": issues}
```

## The takeaway

A statistically significant result isn't automatically a trustworthy one — and it can be wrong for more than one reason at once. Before trusting a test, check who's actually in each group and whether the split between them was fair.
