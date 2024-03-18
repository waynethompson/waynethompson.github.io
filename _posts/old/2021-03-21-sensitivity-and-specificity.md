---
layout: post
title:  "The trade off between Sensitivity and Specificity"
date:   2021-03-21 17:15:00 +1000
categories: [data science, statistics]
tags: [data, statistics]
permalink: /blog/the-trade-off-between-sensitivity-and-specificity/
---

When evaluating a model for a binary dependent variable, for instance logistic regression there is always going to be a trade off between Sensitivity and Specificity.

**Sensitivity** is the proportion of **true positives** identified, that is the proportion of data correctly identified as meeting a certain condition. In the example in the lectures based on credit default it was non-defaulters correctly identified. In a medical study it would be correctly identifying patients with disease.

**Specificity** is the proportion of **true negatives** identified, or the proportion data correctly identified as not meeting a certain criterion. In the lecture example it was the proportion of defaulter correctly identified. Again, in medical research this would be those patients correctly identified as not having the disease.

Sensitivity = TP/TP+ FP

Specificity = TN/ TN+FN

False Positive Rate (FPR) = FP/FP+TN

False Negative Rate (FNR) = FN/ FN+ TP

Sensitivity + False Negative rate = 1

Specificity + False positive Rate = 1

Both are highly desirable, but we cannot have a scenario where we have 100% Specificity and 100% Sensitivity. With Linear Discriminant Analysis (LDA) reducing the posterior probability threshold for identify a true negative will decrease the sensitivity and increase the specificity. The opposite is also true. There will be a point for the threshold where the error rate for false positives and false negatives is at an optimal minimum.

Depending on the use case of the model, a business decision should be made as to what should take precedence. For example, a model that would predict incidence of a fatal disease would give preference to sensitivity over specificity, so that try positives are high and life-threatening false negatives are low. Alternatively, if the model predicted a non-fatal disease for which the treatment had harmful side effects, the sensitivity should be high to ensure the proportion of true negatives is high. If there is no preference, then plotting the error rates and choosing a point where they intersect may be optimal.
