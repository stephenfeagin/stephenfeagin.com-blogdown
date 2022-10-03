---
title: "Project Euler: Multiples of 3 or 5"
subtitle: ""
excerpt: "I've started toying around with Project Euler problems, and I'm starting at problem 1."
date: "2022-10-02"
author: "Stephen Feagin"
draft: false
series:
tags:
  - go
  - r
  - python
  - project euler
  - puzzle
categories:
layout: single # single or single-sidebar
---

I've started toying around with Project Euler problems, and I'm starting at problem 1. The brief reads:

> If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.
> 
> Find the sum of all the multiples of 3 or 5 below 1000.

## Solution

My approach is very straightforward. In Python and R, I can create list/vectors of the multiples of three and five by using `step` or `by` when constructing a sequence. Then it's a simple matter of unioning the two lists (making sure to remove duplicate values for those numbers that are multiples of 15), and getting the sum.

For Go, it was easier to just go through all numbers between 1 and 1000 and check if it's divisible by 3 or 5, keeping a running sum.

{{< panelset >}}
{{< panel name="R" >}}
```r
# First get the multiples of 3 below 1000
threes <- seq(3, 999, by = 3)
# Then the multiples of 5
fives <- seq(5, 995, by = 5)
# Take only the unique values to eliminate those that are multiples of 15
unique_vals <- unique(c(threes, fives))
sum(unique_vals)
```
{{</ panel >}}
{{< panel name="Python" >}}
```python
# Get threes and fives
threes = range(3, 1000, 3)
fives = range(5, 1000, 5)
# To eliminate duplicates, create a set of one and union the other with it
unique_vals = set(threes).union(set(fives))
print(sum(unique_vals))
```
{{</ panel >}}
{{< panel name="Go" >}}
```go
// Variable to hold the running sum
total := 0
// Iterate over values below 1000
for i := 0; i < 1000; i++ {
	// If divisible by 3 or 5, add it to the running total
	if i % 3 == 0 || i % 5 == 0 {
		total += i
	}
}

fmt.Println(total)
```
{{</ panel >}}
{{</ panelset >}}

See all of my Project Euler solutions on [GitHub](https://github.com/stephenfeagin/projecteuler).