---
title: "Project Euler: Sum Square Difference"
subtitle: ""
excerpt: "Find the difference between the square of the sum and the sum of the squares."
date: 2022-10-08
author: "Stephen Feagin"
draft: false
series:
tags:
  - r
  - project euler
  - puzzle
categories:
layout: single # single or single-sidebar
---

Project Euler problem 6: Sum Square Difference. The brief reads:

> The sum of the squares of the first ten natural numbers is,
>
> $$1^2 + 2^2 + ... + 10^2 = 385$$
>
> The square of the sum of the first ten natural numbers is,
>
> $$(1 + 2 + ... + 10)^2 = 3025$$
>
> Hence the difference between the sum of the squares of the first ten natural numbers and the square of the sum is 3025 - 385 = 2640.
>
>Find the difference between the sum of the squares of the first one hundred natural numbers and the square of the sum.

# Solution

The solution for R is actually very easy to do because of R's built-in math functions and out-of-the-box vectorization.

{{< panelset >}}
{{< panel name="R" >}}
R provides "vectorization" out of the box for many functions, and it's generally easy to apply it for functions that don't already have it. Vectorization in this case means the ability to apply a function to every value in a vector, rather than operating on the entire vector as its own object. In R, this is often executed in C or C++, which makes it *fast*. Much faster than writing a loop and iterating over every item in the vector.

This comes into play here because I can write `vec**2` to square every item in the vector, the same that I could write `4**2` to get 16. It doesn't matter that the input is a whole vector instead of a single number, R applies it quickly to every item. At that point, it's trivial to take the sum of the new vector.

```r
vec <- 1:100
print(sum(vec)**2 - sum(vec**2))
```
{{</ panel >}}
{{< panel name="Python" >}}
It's not quite as simple in Python, but still very straightforward. List comprehensions and generator expressions can do the same kind of things that vectorization do in R, and likewise very quickly.
```python
seq = range(1, 101)  # We need to 1:100, not 0:99 which is what range(100) would give
sum_of_squares = sum(i**2 for i in seq)
square_of_sum = sum(seq)**2
print(square_of_sum - sum_of_squares)
```
{{</ panel >}}
{{< panel name="Go" >}}
One of the strengths of Go is that is has very little magic. That means the code has to be more verbose because you do things manually.
```go
// For every number in the slice, add its square to a running total
func sumOfSquares(nums []int) int {
	total := 0
	for _, num := range nums {
		total += num * num
	}
	return total
}

// Add every number in the slice to a running total, then
// return the square of that total.
func squareOfSums(nums []int) int {
	total := 0
	for _, num := range nums {
		total += num
	}
	return total * total
}

func main() {
	// Start by making the sequence of 1 to 100
	seq := make([]int, 100)
	for i := 0; i < 100; i++ {
		seq[i] = i + 1
	}
	// Then take the difference between the two values
	fmt.Println(squareOfSums(seq) - sumOfSquares(seq))
}
```
{{</ panel >}}
{{</ panelset >}}

See all of my Project Euler solutions on [GitHub](https://github.com/stephenfeagin/projecteuler).