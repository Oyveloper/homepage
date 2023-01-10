---
title: "Why Does Regularization Work"
date: 2023-01-10T18:38:21+01:00
draft: false
math: true
tags: ["Machinelearning", "Regularization"]
---

In machine learning we often use regularization in order to combat over- and underfitting. Typically one would use either uidge regularization or lasso regularization, and I assume that you are familiar with these. The explanation for why the regularization helps our models generalize better is typically along the lines of "it simplifies the model", which for me at least is a bit of an unsatisfying answer. Luckily there is one other way you could view it which shows why this works in many cases for ridge regularization.

## Generalization

In order to see why this would give us the outcome we want we should first define what generalization is for a machinelearning model. With a loss functino $l(w, x)$ we define the **model risk** as $R({\bf w}) = \mathbb E [l({ \bf w, x})]$. Further we can define the **empirical risk** $R_S({\bf w}) = \frac 1 m \sum l({ \bf w, x})$ . Finally we get to the **generalization error**

$$
\epsilon _{gen}(w) = R(\mathbf{w}) - R_S(\bf w)
$$

## Stability

Say we have two datasets, $S$ and $S'$. Then we define the **hybrid sample set** $S^{(i)} = \mathbb \{x_1, x_2, ... , x'_i, ..., x_n\}$, i.e. the dataset containing all elements from $S$, but the $i$-th element is replaced with the $i$-th element from $S'$. This lets us define an important concept called **average stability** for a machinelearning algorithm $A$:

$$
\Delta (A) = \mathbb E \left[\frac 1 m \sum_i^m \left(l(A(S), x_i') - l(A(S^{(i)}), x_i')\right)\right]
$$

This equation might seem quite scary, but we can demystify it a bit. The inner subtraction checks the difference in loss between a model not trained on the sample, and one trained on that sample. Note that the only difference between the data used to train the models is that one sample. Then we find the average of this difference over all samples, and find the expected value of this. The useful thing is that it turns out (quite easy to prove) that

$$
\mathbb E [\epsilon_{gen}(A)] = \Delta(A)
$$

(Note here that $\epsilon_{gen}(A)$ simply means $\epsilon_{gen}(\mathbf{w})$ where $\bf w$ is the weights of the algorithm $A$)
This might seem like we did not gain anything, after all we just found out that we could write our nice simple formula for the generalization error in a much more complex way! Rest asured, however, we will relate this to something useful soon. First we need to introduce one more scary formula, namely the **uniform stability**:

$$
\Delta _{sup} = \sup _{S,S'\in (X\times Y)^m} \sup _{i\in [m]} \left|l(A(S), {\bf x}_i') - l(A(S^{(i)}), {\bf x}_i')\right |
$$

The $\sup$ operator is similar to the $\max$, but also works when the set does not contain the maximum itself (think unbounded ranges). This means that the uniform stability creates an upper bound for the average stability, and therefore also our generalization error:

$$
\Delta_{\sup}(A) \ge \Delta(A) = \epsilon_{gen}(A)
$$

## Ah, it's all comming together …

We are almost there, only one theorem left to tie it all together. It turns out that if you minimize the empirical risk, called Empirical Risk Minimization (EMR) (which is what we do when we minimize our loss function), and the loss function is $\alpha$-**strong convex** and $L$-**Lipschitz** then

$$
\Delta_{\sup}(A_{ERM}) \le \frac {4L}{\alpha m}
$$

[1]

Wow that was a mouthful, and a lot of terms. I assume that you have heard of strong convexity and lipschitz, and if not I trust your ability to quickly check wikipedia for a better explanation than I would have given. The upper bound we just found might seem a bit arbitrary, and that's ok since the actual value is not what is important. What is important is that we have now found an upper bound on the generalization error since

$$
\epsilon_{gen}(A) \le \Delta_{\sup}(A)
$$

This is very nice if you happen to have a perfect $\alpha$-strong convex, L-lipschitz function, but as you should know when you do machinelearning, the world is seldom that perfect. Many functions are however convex and lipschitz, and in those cases it is quite easy to construct a new function which meets our criteria. Say we introduce a new term to construct

$$
l'({\bf w, x}) = l({\bf w, x}) + \frac \alpha 2 ||\bf w||^2
$$

This new function is in fact $\alpha$-strong convex, and you might also recognize that this is exactly the form we have when we do ridge regularization. So, to conclude more explicitly, when we do ridge regularization (or L2-regularization) we can make our functions $\alpha$-strong convex, which creates an upper bound on the stability, which in turn bounds the generalization error, which is why we see that our models tend to not over- or underfit to the training data!

## References

[1] O. Bousquet and A. Elisseeff. Stability and generalization. Journal of machine learning research, 2(Mar):  
499–526, 2002.
