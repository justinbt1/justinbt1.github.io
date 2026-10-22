---
title: 'Rounding in Python May Surprise You'
date: 2026-10-22
permalink: /posts/python-rounding
tags:
  - statistics
---
The `round( )` function in Python is used to round a float to the specified precision after the decimal point. For the most part this works as you may expect from high school math classes, rounding to the nearest higher or lower number. It’s when rounding two digits that are exactly between the higher or lower number, so for example rounding 2.5 to an integer, things get interesting.

The most commonly taught arithmetic approach to rounding .5 is to use the round-half-up method and round to the higher number. However Python instead uses bankers rounding, which rounds .5 to the nearest even number.

## The Move to Bankers Rounding
Interestingly, Python 2 originally used the round-half-up rounding method, only adopting the bankers rounding approach during the shift to Python 3. This was change was introduced in PEP 3141 which stated “ float will change in 3.0 to round half toward even”. So what drove this shift in rounding methodology?

While I do not know the exact reason for this decision, adopting bankers rounding brings Python 3 in line with other programming languages such as C, C#, Java and Go. It also means Python now complies with the IEEE 754 standard for how hardware and software handle floating point arithmetic.

## Numerical Stability
There is another very good reason to use bankers rounding for mathematical calculations at scale. As you always round .5 values up, using the round-half-up method at scale can cause a mathematical bias towards the higher number. This introduces a positive skew to the underlying distribution.

For most mathematical and scientific calculations, bankers rounding is actually beneficial because it helps reduce this cumulative rounding bias. However, it can become problematic in situations where users expect traditional “round half up” behaviour or where fairness and transparency are more important than long-term statistical accuracy.

## When is Bankers Rounding an Issue
Bankers rounding can be the cause of subtle bugs in software when there is an expectation gap between the developers and a languages default behaviour. It can also be an issue when dealing with legal, audit, or accounting requirements, which may specify or expect round-one-up rounding to be used.

for an example of a rare case where Bankers rounding may be an issue, we can look at group assignment based on rounded scores. Imagine students are assigned to one of two groups based on the rounded result of an assessment score:

```python
round(2.5)  # 2
round(3.5)  # 4
round(4.5)  # 4
round(5.5)  # 6
```

Suppose students with a rounded score of 5 or higher are placed into an advanced group. A student scoring 4.5 would in Python be rounded down to 4 by default, which may not be inline with the policy. A Python developer unfamiliar with bankers rounding may be unaware that 4.5 does not become 5.

This can lead to situations where individuals perceive the result as unfair, particularly in educational grading, financial reporting, or eligibility thresholds where a single rounded value determines access to a programme, reward, or service. It may not align with business rules, stakeholder expectations or legal requirements for tightly regulated use cases.

For this reason, whenever rounding affects categorisation, entitlements, or other threshold-based outcomes, it is important to explicitly verify that bankers rounding is the desired behaviour. If not, a custom rounding strategy such as round-half-up should be implemented instead.
