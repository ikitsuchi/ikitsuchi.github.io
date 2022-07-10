---
title: 用 Rust 写一个基于 CDCL 算法的 SAT 求解器
date: 2022-07-09T10:50:11+08:00
draft: true
math: true
---

在 OSPP 2022 中我的任务需要用 Rust 写一个 SAT 求解器，写篇博客记录一下。

<!--more-->

## Notation

文字 (literal) 是一个布尔变元 $x_i$ 或其否定 $\neg x_i$。

合取范式 (conjunctive normal form; CNF) 是一个或多个子句 $\omega$ 的合取，用 $\varphi = \bigwedge \limits_{i = 1}^m \omega_i$ 表示。其中每个子句都是文字的析取。一个 CNF 也可以看作是子句的集合。

用 $X = \\{x_1, x_2, \cdots, x_n\\}$ 表示布尔变元的有限集，定义函数 $\nu: X \rightarrow \\{0, u, 1\\}$ 表示对 $X$ 的赋值，其中 $u$ 代表未赋值，$0 < u < 1$，$1 - u = u$。如果所有变元都被赋值为 0 或 1，则称为完全赋值，否则为部分赋值。用 $l^\nu$、$\omega^\nu$、$\varphi^\nu$ 分别表示在相应赋值下文字、子句、CNF 的值：

$$
l^\nu = \begin{cases}
   \nu(x_i) &\text{if } l = x_i \\\
   1 - \nu(x_i) &\text{if } l = \neg x_i
\end{cases} 
$$
$$
\omega^\nu = \max\\{l^\nu\ |\ l \in \omega\\}
$$
$$
\varphi^\nu = \min\\{\omega^\nu\ |\ \omega \in \varphi\\}
$$

布尔可满足性问题 (Boolean satisfiability problem; SAT) 是这样的问题：给定布尔变元的有限集 ，子句集合 `$\{\omega_1, \omega_2, \cdots, \omega_m\}$`，对于给定 CNF 公式 $\varphi = \bigwedge \limits_{i = 1}^m \omega_i$ ，判定是否存在对布尔变元的一组赋值使得该式为真。若存在，则该式是可满足的。

CDCL 算法是建立在 DPLL 算法的基础上的，大体的思想是每次选取一个决策变元，进行回溯搜索，并进行一些优化。

## Boolean Constraint Propagation

我们把除了一个未赋值文字 $l$ 以外其余文字均为假的子句称为单元子句 (unit clause)。想让公式为真，$l$ 必须赋值为真。可以利用这个赋值化简其他子句。这个过程被称为布尔约束传播 (Boolean Constraint Propagation; BCP)。算法会重复执行 BCP，直到没有单元子句。

DPLL 算法的主要优化就是 BCP。伪代码如下：

```
DPLL:
  Run BCP on the formula.
  If the formula evaluates to True, return True.
  If the formula evaluates to False, return False.
  If the formula is still Undecided:
    Choose the next unassigned variable.
    Return (DPLL with that variable True) || (DPLL with that variable False)
```

## Clause Learning

DPLL 有两个缺点：首先它遇到冲突的时候学不到任何东西，其次它每次只会回溯一层。这两点导致它在搜索过程中有很多时间被浪费在了必定失败的搜索空间中。CDCL 利用子句学习优化了 DPLL，为搜索树剪枝。

在 CDCL SAT 求解器中，每个变元 $x_i$ 有以下几个特征：
* value：$\nu(x_i) \in \\{0, u, 1\\}$
* antecedent： $\alpha(x_i) \in \varphi \cup \\{\mathrm{NIL}\\}$
* decision level：$\delta(x_i) \in \\{-1, 0, 1, \cdots, |X|\\}$

若变元 $x_i$ 是子句 $\omega_i$ 用 BCP 推出了赋值，那么称 $\omega_i$ 为 $x_i$ 的 antecedent，$\alpha(x_i) = \omega_i$。如果变元是搜索过程中的决策变元或者是未赋值的，那么它的 antecedent 是 $\mathrm{NIL}$。

变元 $x_i$ 的 decision level 表示了它被赋值时在决策树中的深度。未赋值的变元的 decision level 为 -1，$\delta(x_i) = -1$。被推出的变元 $x_i$ 的 decision level 是其 antecedent 中其他已推出变元的 decision level 的最大值，或者有可能 antecedent 是单元子句，则decision level 为 0。$\delta(x_i) = \max(\\{0\\} \cup \\{\delta(x_j)\ |\ x_j \in \omega \wedge x_j \not ={x_i}\\})$。文字的 decision level 是其变元的 decision level。

用 $x_i = v @ d$ 表示 $\nu(x_i) = v$ 且 $\delta(x_i) = d$。

## Reference

* [Handbook of Satisfiability Chapter 4 Conflict-Driven Clause Learning SAT Solvers](https://www.cs.princeton.edu/~zkincaid/courses/fall18/readings/SATHandbook-CDCL.pdf)