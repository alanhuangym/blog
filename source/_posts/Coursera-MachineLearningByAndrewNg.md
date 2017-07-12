---
title: Coursera Machine Learning Notes
date: 2017/3/15
tags: 
- machine_learning
categories:
- CUHK_Notes
---



[CourseHomePage](https://www.coursera.org/learn/machine-learning/home/)



Week6:

Evaluating a Learning Algorithm

How to improve the algorithm:

What to do next:

1. Get more training examples
2. Try smaller or additional features
3. Try adding ploynomial features (x square or x cube)
4. Try decrease or increse the lamda (regularization parameter)

But every one takes much time. So



### Evaluate the hypothesis:

Divide the dataset to traning set : cross validation set : test set = 6:2:2

Tune the parameters on cross validation set. Minimize the error in CV set. 

Then only apply test on test set.



### Bias VS Variance

High means (underfit and overfit)

1. Get more training examples - fix high variance
2. Try smaller features - fix high variance
3. Try additional features - fix high bias
4. Try adding polynomial features - fix high bias
5. Try decrease lamda - fix high bias
6. Try increase lamda - fix high variance




Precision and Recall

Precision = True Positive/(True positive + False positive) 

= True Pos/Number of Predictive Pos

Recall = True positive /(True positive + False negative)

= True Pos/Number of Actual Pos

one way to balance precision and recall is f1 score

f1 score = 2\*P\*R/(P+R)