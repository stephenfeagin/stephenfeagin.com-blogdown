---
title: Rescaling Variables with R
date: 2016-10-05
status: published
tags: [r, data cleaning]
---

In doing some RA work, I've needed to rescale or normalize different variables. Working with survey data, it can be very
difficult to compare ordinal results across questions. Say for instance that we want to get the correlation of perceived
economic status and perceived social status. The [Latinobar&oacute;metro](http://www.latinobarometro.org/lat.jsp) survey
asks the first question with a scale from 0 to 10. The second item, though, is reported on a five-point scale.
~~If we want to easily interpret correlation, it helps to have these on the same scale.~~<sup>[1](#footnote1)</sup>
Moreover, modeling becomes much easier and more intuitive when we have a simple scale that runs from 0 to 1.

I have also needed to rescale variables from -1 to 1. In particular, using the left-right political spectrum is
difficult when it's scaled 0 to 10. Converting it to \[-1, 1\] puts the very liberal response at -1, the very
conservative response at 1, and the middle at 0. It works much more intuitively, which can be very helpful when
exploring and modeling data.

In other situations, the ordering of two different variables runs in different directions than desired. For instance, in
a question asking how fair the previous election was, the survey responses range from 1 (very fair) to 5 (very unfair).
If we want to look at the correlation of that question with the item asking "How democratic is your country?", we want
the responses to move in the same direction. That is, we should expect a positive correlation between perceived fairness
of the election and perceived democracy in a country. So we need to invert the scaling for the fairness question. This
mainly matters because of the phrasing of the question. If it were asked "How unfair was the last election?", we would
expect high values to indicate greater unfairness. But since the question asked about how fair it was, we intuitively
want the responses to range from 0 (unfair) to 5 (fair).

I've written a handful of functions that help out with these tasks. They're nothing groundbreaking, but I've used them
over and over again in this type of work. Below is the [gist](http://gist.github.com/discover) with that code. Hope it
comes in handy for someone!

<script src="https://gist.github.com/stephenfeagin/248154dfe4ed0c0bbe0f3b92725bf7da.js"></script>

<a name="footnote1" id="footnote1">1</a>: A very helpful
[reddit user](https://www.reddit.com/r/rstats/comments/562owx/rescaling_variables_with_r_just_a_short_blog_post/d8hefet)
pointed out to me that this is, of course, false. Correlation is always scaled from -1 to 1, no matter how the original
variables were scaled. I was clearly being loose with my language and not paying attention to the differences between
colloquial and technical uses of terms like correlation.
