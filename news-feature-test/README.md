# New Landing Page Test — Does It Actually Work Better?

## The situation

A news site is testing a new landing page design against the old one. 100 users were split evenly — 50 saw the old page, 50 saw the new one — and the team wants a clear answer on two things: does the new page keep people on the site longer, and does it convert better? They also wanted to know if language preference (English, French, Spanish) affects any of this.

## Question 1: Do people spend more time on the new page?

```python
old_page = df[df['landing_page'] == 'old']['time_spent_on_the_page']
new_page = df[df['landing_page'] == 'new']['time_spent_on_the_page']
stat, p_value = stats.ttest_ind(new_page, old_page, alternative='greater')
```

- Old page average: 4.53 minutes
- New page average: 6.22 minutes
- t = 3.79, **p = 0.0001**

Yes. People spend almost 2 minutes longer on the new page, and it's very unlikely to be chance.

## Question 2: Does the new page convert better?

```python
conversions = [33, 21]  # new, old
nobs = [50, 50]
stat, p_value = proportions_ztest(conversions, nobs, alternative='larger')
```

- Old page conversion: 42%
- New page conversion: 66%
- z = 2.41, **p = 0.008**

Yes, clearly. A 24-point jump in conversion rate, and statistically significant.

## Question 3: Does language affect conversion?

Before trusting the headline result, it's worth checking whether something like language is quietly driving the numbers — the same kind of check that mattered in the Spanish translation case study.

```python
contingency = pd.crosstab(df['language_preferred'], df['converted'])
stat, p_value, dof, expected = chi2_contingency(contingency)
```

Result: chi-square = 3.09, **p = 0.21**. Not significant.

English speakers did convert at a slightly higher rate in this sample, but with only ~32-34 users per language, that gap is well within what random noise could produce. I didn't treat it as a real effect just because it looked interesting.

## Question 4: Does time spent on the new page vary by language?

```python
english = new_page_df[new_page_df['language_preferred'] == 'English']['time_spent_on_the_page']
french = new_page_df[new_page_df['language_preferred'] == 'French']['time_spent_on_the_page']
spanish = new_page_df[new_page_df['language_preferred'] == 'Spanish']['time_spent_on_the_page']
stat, p_value = f_oneway(english, french, spanish)
```

Result: F = 0.85, **p = 0.43**. Not significant — English, French, and Spanish users spend roughly the same amount of time on the new page.

## One more check before trusting the result

Since the earlier Spanish translation case study showed how an uneven group makeup can quietly wreck a result, I checked whether language was evenly spread across control and treatment before accepting these numbers:

```
control:    Spanish 17, French 17, English 16
treatment:  Spanish 17, French 17, English 16
```

Perfectly balanced. No hidden imbalance skewing the result this time.

## Final recommendation

**Launch the new landing page.** Users spend meaningfully more time on it (6.22 vs 4.53 minutes) and convert at a much higher rate (66% vs 42%), and both results are statistically solid. Language preference doesn't meaningfully affect conversion or time spent, and the groups were evenly balanced by language — so there's no hidden confound behind these numbers. This is a clean result to act on.

## The takeaway

A good result isn't just about finding a significant p-value — it's about checking there isn't a quieter factor (like language, or country, in the Spanish case study) sitting underneath it that would change the story. Here, that check came back clean, which makes the recommendation genuinely trustworthy.

