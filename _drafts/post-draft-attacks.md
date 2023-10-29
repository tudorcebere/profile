Let’s get a basic differentially private mechanism, instantiate an attacker against it and see how it behaves! This will give us a neat understanding of why we care about Differential Privacy.

## Analyzing the randomized mechanism for counting

 A counting query is an algorithm that takes database $D$ and outputs the fraction of elements that satisfy a predicate $q$.  The randomized mechanism is a $\epsilon$-DP algorithm used for counting queries defined as:

$$
M_{count}(D) = \overset{|D|}{ \underset{i}{\sum}} q(D[i])
$$

$$
q(x) = \begin{cases} \lnot q(x), \text{with probability } (1 - p)/2 \\  q(x), \text{with probability } (1 + p)/2 \end{cases}
$$

This mechanism is  $ln\frac{1 - p}{1 + p}$-DP. Let's implement this in plain Python for a cardinality algorithm. Namely, we would like to estimate the length of the database in a differentially private way:


```python
import random

def private_count(D: list, p: float) -> int:
  count_result = 0
  for i in range(len(D)):
    sample = random.random()
    if sample < (1 + p)/2:
      count_result += 1
  return count_result
```

Let’s create a simple attacker against this query type and see how the ROC curve and AUC score evolves with the epsilon. The distinguishing function of our attacker will be to always to pick the dataset with the closest length as the given output of the mechanism.  Remember that $\epsilon$-DP should work against any underlying data or attacker, so we should try as hard as possible to create a degenerate edge case! 

```python
def attack(D1: list, D2: list, out: int) -> int:
   diff_1 = abs(len(D1) - out)
   diff_2 = abs(len(D2) - out)
   return 1 if diff_1 < diff_2 else 2
```

```python
D1 = [1] # this datasets length is 1
D2 = [1, 2] # this datasets length is 2
number_of_experiments = 10000
p = 0.2

for i in range(number_of_experiments):
   O, b = private_count(D1, D2, p)
   b_prime = attack(D1, D2, O)
   # ... store results and plot ROC curve/compute AUC ...
   # (the full code is available at the end of the blogpost
```

![roc_auc_attacker.png](https://github.com/tudorcebere/profile/blob/master/files/blog_resources/dp_ht/roc_auc_attacker.png?raw=true)

The ideal degenerate case is when there are a few samples to distinguish from! We will evaluate the effectiveness of this attack on a fixed very small length, in which the errors are big and the distinguishing power is significant. Our attacker was crafted just for the sake of evaluating how the performance of the adversary degrades based on the epsilon. Note that this is probably not the strongest attacker, but it is good enough to show the performance on an edge case! There might be way stronger attackers that we don’t know how to create, but $\epsilon$-DP has to bound the performance of all of them.

An impressive property is that the attacker bounding properties holds for all algorithms that satisfy $\epsilon$-DP, for example, the [Laplacian mechanism](https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf) or the [Exponential mechanism](https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf). Now, let’s see analyse how one of the most well-known relaxations of $\epsilon$-DP behaves from an attacker's point of view!