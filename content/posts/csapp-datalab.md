---
title: CS:APP Data Lab
date: 2021-08-20T19:38:14+08:00
draft: true
tags:
  - CS:APP
categories:
  - Notes
  - Computer System
---

[GitHub repo](https://github.com/ikitsuchi/CSAPP-labs-solutions/tree/master/data)

<!--more-->

## bitXor

> x^y using only ~ and &.

根据德摩根律，`x ^ y = ~((x & y) || (~x & ~y)) = ~(x & y) & ~(~x & ~y)`。

```cpp
int bitXor(int x, int y) {
  return ~(x & y) & ~(~x & ~y);
}
```

## tmin

> Return minimum two's complement integer.

```cpp
int tmin(void) {
  return (1 << 31);
}
```

## isTmax

> Returns 1 if x is the maximum, two's complement number, and 0 otherwise.

如果 `x` 是 tmax, 那么 `x + (x + 1) = -1`. 然而 -1 也满足这个性质，所以用 `-1 + 1 = 0` 把 -1 去掉。

```cpp
int isTmax(int x) {
  return !(~((x + 1) + x)) ^ !(x + 1);
}
```

## allOddBits

> Return 1 if all odd-numbered bits in word set to 1. (where bits are numbered from 0 (least significant) to 31 (most significant))

可以用一个 mask `0xAAAAAAAA` 去判断 x 是否合法。

```cpp
int allOddBits(int x) {
  int mask = 0xAA + (0xAA << 8);
  mask = mask + (mask << 16);
  return !((x & mask) ^ mask);
}
```

## negate

> Return -x.

根据补码的定义。

```cpp
int negate(int x) {
  return (~x + 1);
}
```

## isAsciiDigit

> Return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9').

首先判断是否为 0x3X，再判断第 4 个 bit 是否为 0，若为 1 则判断第 2 和 3 bit 是否为 0，每个答案都是 0 / 1，用位运算统计起来。

```cpp
int isAsciiDigit(int x) {
  int lead = !((x >> 4) ^ 3);
  int bit3 = !(x & 8);
  int bit12 = !(x & 6);
  return lead & (bit3 | (bit12 ^ bit3));
}
```

## conditional

> Same as x ? y : z.

把 x 先直接转化为 0 或 1，再取补码，1 能变成全 1，这样 x 就变成了一个 mask。

若 x 为全 1，x & y = y，若 x 为全 0，x & y = 0。z 同理，反过来即可。

```cpp
int conditional(int x, int y, int z) {
  x = !!x;
  x = ~x + 1;
  return (x & y) | (~x & z);
}
```

## isLessOrEqual

> If x <= y then return 1, else return 0.

$x \le y \Rightarrow y - x \ge 0$.

那么将 x 取补码得到 -x 与 y 相加得到 y - x，判断符号位。

但如果溢出了，这个判断方式就挂掉了。若 x、y 同号，必然不会溢出。而 x、y 不同号，就判断一下是不是 x 是负数，y 是正数。

还有一个问题是如果 x 是 tmin，取补码并得不到 -x。当 x 为 tmin 不等式结果必为 1，考虑 y。若 y >= 0，符号就判定了；若 y < 0，加上 $2^31$ 后结果 > 0，答案也对。所以好像不用管（

```cpp
int isLessOrEqual(int x, int y) {
  int sum = (~x + 1) + y;
  int sign_x = (x >> 31) & 1;
  int sign_y = (y >> 31) & 1;
  int sign_minus = ((sum >> 31) & 1);
  return (sign_x & !sign_y) | ((sign_x ^ sign_y) & sign_minus);
}
```

## logicalNeg

> Implement the ! operator, using all of the legal operators except !.

对一个除了 0 和 tmin 以外的数，它的补码是它的相反数，tmin 和 0 的补码还是本身，可以用补码和原码的符号位都是 0 来判断。

```cpp
int logicalNeg(int x) {
  return ((((x | (~x + 1)) >> 31) & 1) ^ 1);
}
```

## howManyBits

> Return the minimum number of bits required to represent x in two's complement.

如果是正数，即为最高位 1 所在位 + 1 位（符号位）。

如果是负数，那就直接取反，还是找最高位 1。

```cpp
int howManyBits(int x) {
  int sign = (x >> 31) & 1;
  x = x ^ (~sign + 1);
  int bin_1 = 0, bin_2 = 0, bin_4 = 0, bin_8 = 0, bin_16 = 0;
  bin_16 = !!(x >> 16) << 4;
  x = x >> bin_16;
  bin_8 = !!(x >> 8) << 3;
  x = x >> bin_8;
  bin_4 = !!(x >> 4) << 2;
  x = x >> bin_4;
  bin_2 = !!(x >> 2) << 1;
  x = x >> bin_2;
  bin_1 = !!(x >> 1);
  x = x >> bin_1;
  return bin_16 + bin_8 + bin_4 + bin_2 + bin_1 + x + 1;
}
```

## floatScale2

> Return bit-level equivalent of expression 2*f for floating point argument f. Both the argument and result are passed as unsigned int's, but they are to be interpreted as the bit-level representation of single-precision floating point values. When argument is NaN, return argument.

先把 0、INF、NaN 特判掉。

根据 IEEE 浮点表示，$V=(-1)^{s}\times{M}\times{2^{E}}$。分类讨论 normalized 和 denormalized 。

对于 normalized ，$E = e - bias$，$M = 1 + f$。那么我们只需要让 $e + 1$ 。但倘若 $e + 1$ 得到溢出的结果，应该返回一个 INF。

对于 denormalized ，$E = 1 - bias$，$M = f$。直接令 $f \times{2}$ 即可。但又存在一个问题，如果 $f$ 溢出了呢？这个思考一下发现并没有关系。因为这个溢出一定会导致 $e$ 从 0 变成 1，但由于 $E$ 的定义，它的值不变。这也就是书中所说的 "It provides for smooth transition from denormalized to normalized numbers" 。但换言之，这部分精度的损失是不可避免的。

```cpp
unsigned floatScale2(unsigned uf) {
  int s = (uf >> 31) & 1;
  int e = (uf >> 23) & 0xFF;
  int f = uf & 0x7FFFFF;
  int INF = (0xFF << 23);
  if (e == 0xFF) return uf; // INF / NaN
  if (!e && !f) return uf; // +0 / -0
  if (e) {
    if (e + 1 == 0xFF) return (INF | (s << 31));
    else return ((s << 31) + ((e + 1) << 23) + f);
  } else {
    return ((s << 31) + (e << 23) + (f << 1));
  }
}
```

## floatFloat2Int

> Return bit-level equivalent of expression (int) f for floating point argument f.

先特判 NaN 和 INF。

如果 $E < 0$，那么这个数绝对值就小于 1，直接 return 0 即可。这个情况包含了所有 denormalized number 和 0 。

剩下的只有一些 normalized ，根据书可以直接看作二进制的科学记数法然后移位。记得要加上被丢弃的 1 。

注意上界：$E \le 31$。但有特殊情况：如果是负数是可以取到 $-2^{-31}$ 的，所以要再特判一下。

```cpp
int floatFloat2Int(unsigned uf) {
  int s = (uf >> 31) & 1;
  int e = (uf >> 23) & 0xFF;
  int f = uf & 0x7FFFFF;
  int bias = (1 << 7) - 1;
  int E = e - bias;
  if (e >= 0xFF) return 0x80000000u;
  if (E < 0) return 0;
  if (E >= 31) {
    if (E == 31 && s && !f) return -0x80000000;
    else return 0x80000000u;
  }
  f = ((f | (1 << 23)) >> (23 - E));
  if (s) return -f;
  else return f;
}
```

## floatPower2

> Return bit-level equivalent of the expression 2.0^x (2.0 raised to the power x) for any 32-bit integer x.

分类讨论 $e = bias + x$。

*  $e > 0$，判断是不是 INF，然后移位就好了。

*  $e \le 0$，需要判断能否用 denorm 表示。



```cpp
unsigned floatPower2(int x) {
  int e = (1 << 7) - 1 + x;
  if (e > 0) {
    if (e >= 0xFF) return (0xFF << 23);
    else return (e << 23);
  } else {
    if (x < -149) return 0;
    else {
      return (1 << (149 + x));
    }
  }
}
```