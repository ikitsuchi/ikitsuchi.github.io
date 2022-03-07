---
title: "UCB CS 61A Notes"
date: 2022-03-06T16:05:12+08:00
draft: false
categories:
  - Notes
  - Computer Science
tags:
  - CS
---

This post is the notes I took while learning CS 61A (Spring 2022).

<!--more-->

# Functions, Expressions, Values

* Programs work by manipulating **values**, each value has a **data type**.
* **Expressions** in programs evaluate to values.
* **Functions** encapsulate a series of statements that maps **arguments** to a **return value**.
* A **def statement** creates a function object with certain **parameters** and a body and binds it to name in the current environment.
* A **call expression** creates a new **frame** in the environment, binds the function call's arguments to the parameters in that frame, then executes the body of the function in the new frame.

# Control

* A **side effect** is when something happens as a result of calling a function besides just returning a value.
* Conditionals, booleans, iteration...

# Higher-Order Functions

## Designing functions

* A pure function's **behavior** is the relationship it creates between input and output.
* Give each funtion exactly one job, but make it apply to many related situations.
* **Don't Repeat Yourself (DRY)**: Implement a process just once, execute it many times.

## Higher-order functions

A higher-order function is a function that either:

* Takes another function as an argument
* Returns a function as its result

All other functions are considered first-order functions.

## Lambda expressions

A **lambda expression** is a simple function definition that evaluates to a function.

The syntax: `lambda <parameters>: <expression>`

## Conditional expressions

A conditional expression has the form:

`<consequent> if <predicate> else <alternative>`

# Environments

## Multiple environments

![multiple environments](/images/cs61a/environments_multiple_environments_0.png)

* Every expression is evaluated in the context of an environment.
* A name evaluates to the value bound to that name in the earlist frame of the current environment in which that name is found.

## Environments for higher-order functions

![multiple environments](/images/cs61a/environments_multiple_environments_1.png)

**Functions are first class**: Functions are treated as values in Python.

## Environments for nested def statements

![multiple environments](/images/cs61a/environments_multiple_environments_2.png)

* Every user-defined function has a parent frame.
* The parent of a function is the **frame in which it was defined**.
* Every local frame has a parent frame.
* The parent of a frame is the **parent of the called function**.

## Function currying

**Currying**: Converting a function that takes multiple arguments into a single-argument higher-order function.

Here is a function that currys any two-argument function:

```python
def curry2(f):
  def g(x):
    def h(y):
      return f(x, y)
    return h
  return g
```

```python
curry2 = lambda f: lambda x: lambda y: f(x, y)
```

```python
make_adder = curry2(add)
make_adder(2)(3)
```