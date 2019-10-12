---
layout: post
title:  " HW公司校园招聘考试题目一题小计"
date:   2019-10-12 16:30:00
excerpt: ""
author: Chi Shen
tags: [R, Primary num, Examination]
feature: https://upload.wikimedia.org/wikipedia/commons/0/0d/Iss007e10807.jpg
comments: true

---

**Chi Shen, Oct 12 2019**

## 题目描述

> 质数的定义为：在大于1的自然数中，除了1和它本身，不能整除其他自然数的数，如2，3，23等。
>
> 给定一个取值范围[low, high]，找到该范围内的所有质数，输出这些质数的十位数之和与个位数之和的较小值，如果该质数小于10，则其十位数为0，注意low在取值范围内，high不在。

## 输入描述

> 第一行输入：low，high，用例保证均不小于0，且low < high，不用考虑输入合法性，如果没有质数，则输出0。

## 输出描述

> 输出找到的质数和十位数之和与个位数之和的较小值。

## 解题思路

```R

# Rversion: 3.5.3

lower <- as.numeric(readline('Step 1：Please input a smaller integer value larger than 1:'))
upper <- as.numeric(readline('Step 2：Please input a larger integer value larger than 1:'))

if (lower >= upper) {

	print('Please make sure the second value is larger than first value！')

} else {

	sequence <- seq(lower, upper - 1, 1)

	#---------------判断质数------------------#
	x <- vector(mode = 'logical', length = upper - lower)


	for (i in 1:length(x)) {
		if (all(sequence[i] %% (2:sqrt(sequence[i])) != 0)) {
			x[i] <- TRUE
		} else {
			x[i] <- FALSE
		}
	}

	prime_num <- sequence[x]

	if (length(prime_num) == 0) {

		sprintf('There is no prime number between %i and %i', lower, upper)

	} else {

		#---------------取个位数和十位数------------------#
		ten_dig <- vector(mode = 'numeric', length = length(prime_num))
		sin_dig <- vector(mode = 'numeric', length = length(prime_num))

		for (j in 1:length(prime_num)) {
			ten_dig[j] <- prime_num[j] %/% 10 %% 10 #取除数再取余数
			sin_dig[j] <- prime_num[j] %% 10 #取余数
		}

		#---------------分别求个位数和十位数之和------------------#
		ten_sum <- sum(ten_dig)
		sin_sum <- sum(sin_dig)

		sprintf('The small value of summarize of ten digits (%i) and summarize of single digits (%i) in prime number in [%i，%i）is: %i',
				ten_sum, sin_sum, lower, upper, min(c(ten_sum, sin_sum)))
	}	
}

```

