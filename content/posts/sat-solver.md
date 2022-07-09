---
title: 用 Rust 写一个基于 CDCL 算法的 SAT 求解器
date: 2022-07-09T10:50:11+08:00
draft: true
math: true
---

在 OSPP 2022 中我的任务需要用 Rust 写一个 SAT 求解器，写篇博客记录一下。

<!--more-->

# Definition

文字 (literal) 是一个布尔变元或其否定。

合取范式 (conjunctive normal form; CNF) 是一个或多个子句的合取，其中每个子句都是文字的析取。

布尔可满足性问题 (Boolean satisfiability problem; SAT) 是这样的问题: 给定布尔变元集合 `$\{x_1, x_2, \cdots, x_n\}$`，子句集合 `$\{c_1, c_2, \cdots, c_m\}$`，对于给定 CNF 公式 $F = \bigwedge \limits_{i = 1}^m c_i$ ，判定是否存在对布尔变元的一组赋值使得该式为真。若存在，则该式是可满足的。

# Algorithm

我们要使用的算法是冲突驱动的子句学习算法 (Conflict-driven clause learning; CDCL)，它主要包含了对朴素算法的以下几种优化：

## Boolean Constraint Propagation

我们把除了一个未赋值文字 $\mathrm{L}$ 以外其余文字均为假的子句称为单元子句 (unit clause)。想让公式为真，$\mathrm{L}$ 必须赋值为真。之后就可以化简公式，对于剩余每个子句 $\mathrm{C}$:
* 若 $\mathrm{C}$ 包含 $\mathrm{L}$，则删除 $\mathrm{C}$；
* 若 $\mathrm{C}$ 包含 $\neg\mathrm{L}$，则删除 $\neg\mathrm{L}$。

这个过程被称为布尔约束传播 (Boolean Constraint Propagation; BCP)。算法会重复执行 BCP，直到没有单元子句。

## Clause Learning

朴素算法的一个缺陷是它在遇到冲突时学不到新的东西。事实上我们可以通过遇到的冲突添加新的约束，为搜索剪枝。

```
DPLL:
  Run BCP on the formula.
  If the formula evaluates to True, return True.
  If the formula evaluates to False, return False.
  If the formula is still Undecided:
    Choose the next unassigned variable.
    Return (DPLL with that variable True) || (DPLL with that variable False)
```

# Reference

* [Conflict Driven Clause Learning - University of Washington | CSE 442](https://cse442-17f.github.io/Conflict-Driven-Clause-Learning/)
  
  里面的可视化对理解 clause learning 很有帮助！

* [Handbook of Satisfiability Chapter 4 Conflict-Driven Clause Learning SAT Solvers](https://www.cs.princeton.edu/~zkincaid/courses/fall18/readings/SATHandbook-CDCL.pdf)