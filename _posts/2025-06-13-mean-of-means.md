---
title: 'Is the average of averages the same as the overall average?'
date: 2025-06-13
permalink: /posts/is-the-average-of-averages-the-same-as-the-overall-average
tags:
  - data science
  - statistics
---
There is a simple question that always seems to crop up when discussing everything from analytics dashboards to parallel computation. That is, when you have multiple groups of data (e.g., from different sources or processes), each with its own average, can you simply average those individual averages to get the true average across all the data?

The short answer to this question is no, the longer answer is it depends. The average of multiple averages over sets of elements is only guaranteed to be the same as the average of all elements when:

- The cardinality (number of elements) of each averaged set are the same.
- The trivial and rare case where all set averages equal to zero.

Let's have a look at why, the mean of the values in the below set is 5.5:

$$
a = \{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 \}
$$

However, if we break this set into three arbitrary sets $(a, b, c)$ and take the mean of each we get a combined average across all sets of 5.17:

$$
a = \{1, 2, 3\}
$$  

$$
b = \{4, 5, 6\}
$$  

$$
c = \{7, 8, 9, 10\}
$$  

$$
(\bar{a} + \bar{b} + \bar{c}) / 3 = 5.17
$$

The discrepancy is due to the combined average not accounting for the number of elements in each set. This discrepancy will become larger as the difference between the set sizes increases.

The discrepancy is due to the combined average not accounting for the number of elements in each set. This discrepancy will become larger as the difference between the set sizes increases.

- For each set calculate a weight, this can be found by dividing the number of elements in a set by the total number of elements in all of the sets combined.
- Weight the average of each set by finding the product between the sets average value and the associated weight.
- Sum each of the weighted averages to give a combined average that represents the true average value of all elements in all sets.

We can apply the above process to our sets (a, b, c) to calculate the weighted averages, which when summed these give the true overall average of 5.5.

$$
\bar{a}\ (3 / 10) = 0.6
$$  

$$
\bar{b}\ (3 / 10) = 1.5
$$  

$$
\bar{c}\ (4 / 10) = 3.4
$$  

$$
0.6 + 1.5 + 3.4 = 5.5
$$
