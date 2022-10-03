---
title: ioslides with R Markdown
date: 2016-10-03
status: published
tags: [rstudio, r markdown, html5, ggplot2, teaching, r]
---

Over the weekend, I decided to write up some slides for my stats section on Monday. Having really gotten into the groove
of using R Markdown, and having gotten a little rusty on using LaTeX with **knitr**, I decided to play around with HTML5
slides instead of going the beamer route. In the spirit of results first, the finished presentation is shown below.
Click inside the frame, then use your left and right arrows to navigate.

<iframe src="plsc_500_section_5_slides.html"
        width=120% 
        height=750x
        webkitallowfullscreen
        mozallowfullscreen
        allowfullscreen></iframe>

I was at first very interested in using [reveal.js](https://github.com/hakimel/reveal.js), as I really liked the
multi-dimensional aspect of the presentation. I had a code chunk that demonstrated using `stat_function()` in
**ggplot2**, and included a hexidecimal color value. I wanted to have a brief aside where I could discuss hex colors
without putting it right in the path of the rest of the rest of the presentation. I know this was a bit of a tangent
from the main discussion, and it wasn't super relevant, but I wanted to have the information in there anyway. In case
someone asked about it (I tell myself).

Using the **revealjs** package, I created a new R Markdown file that would output as a reveal.js presentation. I looked
up the documentation on
[how to do everything within RStudio](http://rmarkdown.rstudio.com/revealjs_presentation_format.html), and got going.
However, I quickly ran into a problem. When using RStudio, reveal.js only allows for vertical arrangement of slides
_within sections_. That is to say, the presentation would be organized with columns of slides, each belonging to the
same section. The trouble is that section heading slides only contain the name of the section, and no further
information. So it's not possible, as far as I could figure out, to create a new slide below my **ggplot2** slide that
would talk about hexidecimal colors, as the slide on the "main" level could only show the section name, not the full
slide contents. Back to the drawing board.

I experimented with [DZSlides](http://paulrouget.com/dzslides/) and [Slidy](https://www.w3.org/Talks/Tools/Slidy2/), but
ultimately decided on [ioslides](http://rmarkdown.rstudio.com/ioslides_presentation_format.html) because it seemed that
RStudio had pretty good support and documentation for that format. For one, it doesn't require using command line pandoc
in order to write into the slide format, which is very helpful. ioslides doesn't feature two-dimensional presentation,
and it's not incredibly customizable, but it looks really good right out of the box.

The main customization I ended up using frequently is `{.build}`. Placed immediately after a new slide title, this
command prints each element of the slide separately, one after another. Using `incremental: true` in the YAML header
does this for ordered and unordered lists, but I wanted to make sure that my R code chunks would appear at the right
times as well. Without `{.build}`, the slide would appear without text, but with the code chunks already printed.
Unfortunately, I could not work out a way to declare the `{.build}` attribute for every slide. I tried putting
`build: true` in the YAML header (similar to using `smaller: true` in the header rather than `{.smaller}` for every
slide), but it didn't work. So I have the attribute listed at the head of all slides that are not just showing plots
(where it doesn't really matter if things are loaded one at a time).

Beyond being mindful of what fits on one slide and the `{.build}` attribute, creating the ioslides presentation in
RStudio was really not much different from creating any other R Markdown document. And, being HTML, it knitted much
faster, enabling easier and quicker previewing of slides.

I'd say that the section went over well, and I certainly felt much more comfortable working through presentation slides
instead of raw R script. It gave the section much more structure, and required me to spend more time thinking about the
material we were going to cover and how best to present it. Hopefully that this post is useful for those considering
writing presentation slides in RStudio.
