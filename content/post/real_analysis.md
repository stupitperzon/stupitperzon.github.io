---
date: "2024-03-28"
tags: ["analysis", "math"]
title: "Real Analysis"
---

## 1.3 - The Axiom of Completeness

The least upper bound is the supremum. The greatest lower bound is the infimum.

A supremum \\( s = \sup A \\) of \\( A \subseteq \mathbb{R} \\) must satisfy:
1. \\( a \leq s \forall a \in A \\) or \\( s \\) is an upper bound of \\( A \\)
2. For any upper bound \\( b \\) of \\( A \\), \\( b \geq s \\)

The axiom of completeness states that every bounded, non-empty set of numbers has a supremum in \\( R \\).

A supremum can also be expressed as \\( s = \sup A \\) iff \\( \forall \epsilon > 0 \exists a \in A \mid s - \epsilon < a \\). To prove,
1. Assume the former. \\( s - \epsilon < s \\), which means \\( s - \epsilon \\) is no longer an upper bound. So therefore, there exists an \\( a \in A \\) which is greater than \\( s - \epsilon \\).
2. Assume the latter. If there are any other upper bounds \\( b < s \\), then they are not upper bounds of \\( A \\). Therefore, the former is true.

## 1.4 Conesquences of Completeness

The Archimedean Property states:
1. Given any number \\( x \in \mathbb{R} \\), there exists an \\( n \in \mathbb{B} \\) satisfying \\( n > x \\).
2. Given any real number \\( y > 0 \\), there exists an \\( n\in \mathbb{B} \\) satisfying \\( 1/n < y \\).

For any two real numbers, there exists a rational number in between them. Let the two real numbers be \\(a, b\\) and let the rational number be \\(r = \frac{m}{n}\\). Then, we pick an \\( n\in \mathbb{N} \\) such that \\(\frac{1}{n} < b - a\\). Since \\( a < \frac{m}{n} < b \\), then \\( an < m < nb \\), and \\( m \\) must be chosen as the least natural number satisfying the prior, namely \\( m - 1 \le na < m \\).

\\( na < m \\) so what is left is to prove \\( m < nb \\). Since \\( b > a + \frac{1}{n} \\), then \\( m < n(a + \frac{1}{n}) \leq na + 1 \\) which matches with our prior constraints of \\( m \\).

To prove that there exists a real number \\( \alpha\in \mathbb{R} \\) satisfying \\( \alpha^2 = 2 \\), consider \\( T = \\{ t\in \mathbb{R} : t^2 < 2\\} \\) and prove that \\( \sup T = \sqrt{2} \\) and not \\( \sup T < \sqrt{2} \\) or \\( \sup T > \sqrt{2} \\). If \\( \sup T < \sqrt{2} \\), then there must be some element \\( n_0 \in \mathbb{N} \\) such that \\( \sup T + \frac{1}{n_0} < \sqrt{2} \\), but \\( \sup T + \frac{1}{n_0} \in T \\), contradicting the definition of supremum. The rigor comes from finding the exact value of \\( n_0 \\).

The cardinalities of \\( \mathbb{N} \\) and \\( \mathbb{Z} \\) are the same. This seems weird because \\( \mathbb{N} \\) is obviously smaller than \\( \mathbb{Z} \\) since it's the nonzero positive integers. But, \\( f(n) = (n-1)/2 \\) if \\( n \\) is odd, and \\( f(n) = -n/2 \\) if \\( n \\) is even shows a 1-1 and onto correspondence between \\( \mathbb{Z} \\) and \\( \mathbb{N} \\). Countable sets contain a bijection to the natural numbers. Uncountable sets do not.

\\( \mathbb{Q} \\) is countable. To see why, consider \\( A_n = \\{ \frac{p}{q} \mid p + q = n \\} \\) and flatten the sets and assign an index to each element. We see this mapping corresponds to the natural numbers and eventually all rational numbers are listed out.

\\( \mathbb{R} \\) is not countable. The Nested Interval Property states that when \\( I_n = [a_n, b_n] = \\{ x \in \mathbb{R} : a_n \le x \le b_n \\} \\), \\( \bigcap_{n=1}^{\infty} I_n \neq \emptyset \\). We can see at the minimum \\( \sup I_n \\) is in the intersection due the axiom of completeness. But the Nested Interval Property need not to include a rational number in the intersection. Similarly, the irrational numbers are not countable, because the reals are not countable, and the reals are the union of the irrationals and the quotients, and the quotients are countable.

## 1.5 Cantor's Theorem

\\( (0, 1) \subseteq \mathbb{R} \\) is uncountable. To see this, define a mapping from the natural numbers to the reals with \\( f(i) = 0.a_{i1}a_{i2}... \\). We can find a decimal that is not present in this mapping by finding a decimal \\(0.b_{1}b_{2}...\\) where \\( b_n = 2 \\) if \\( a_{nn} \neq 2 \\) and \\( b_n = 3 \\) if \\( a_{nn} = 2 \\).

Cantor's Theorem is given any set \\( A \\), there doesn't exist a function \\( f : A \rightarrow P(A) \\) that is onto, where \\( P(A) \\) is the power set function, or the set of all subsets of \\( A \\).

The countable sets are in one equivalent class (the same countability). The reals are in a different one. Then the power set of the reals are in a different one from the reals.

The cardinality of the natural numbers is \\( \aleph_0 \\), which is the smallest infinite cardinality. The cardinality of the real numbers is \\( c \\). It was proved by Godel that whether there is a cardinality in between \\( \aleph_0 \\) and \\( c \\) is undecidable.

## Resources

<a id="1">[1]</a>
Understanding Analysis - Stephen Abbott