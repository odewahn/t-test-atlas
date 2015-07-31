In every data analysis, you have to string together many tools. You need tools for data wrangling, visualisation, and modelling to understand what’s going on in your data. To use these tools effectively, you need to be able to easily flow from one tool to the next, focusing on asking and answering questions of the data, not struggling to jam the output from one function into the format needed for the next. Wouldn’t it be nice if the world worked this way! I spend a lot of my time thinking about this problem, and how to make the process of data analysis as fast, effective, and expressive as possible. Today, I want to show you a new technique that I’m particularly excited about.

R, at its heart, is a functional programming language: you do data analysis in R by composing functions. However, the problem with function composition is that a lot of it makes for hard-to-read code. For example, here’s some R code that wrangles flight delay data from New York City in 2013\. What does it do?

<pre data-code-language="python" data-executable="true" data-type="programlisting">
library(nycflights13)
library(dplyr)

head(
  arrange(
    summarise(
      group_by(
        filter(
          flights,
          !is.na(dep_delay)
        ),
        year, month, day
      ),
      delay = mean(dep_delay)
    ),
    desc(delay)
  ),
  5
)
</pre>

It’s hard to read this code because:

1.  The innermost operation is performed first, so to read the operations in sequence you have to read from inside-out, from right-to-left.
2.  Function arguments become increasingly spread apart (this is known as the [Dagwood sandwich](http://en.wikipedia.org/wiki/Dagwood_sandwich) problem).

We can make the code easier to read by using a new operator: `%>%`, the pipe, provided by the [magrittr](https://github.com/smbache/magrittr) package. It turns function composition into a sequence of imperative commands: “Start here, then do this, then do that, then do something else.” Here’s what the previous code looks like if we use `%>%` (when reading, pronounce `%>%` as “then”):

<pre data-code-language="python" data-executable="true" data-type="programlisting">
flights %>%
  filter(!is.na(dep_delay)) %>%
  group_by(year, month, day) %>%
  summarise(delay = mean(dep_delay)) %>%
  arrange(desc(delay)) %>%
  head(5)
</pre>

Even if you’ve never used R before, you should be able to make sense of this: we start with flights data, then remove all flights with missing departure delay. Next we group by date, and summarize to work out the average daily delay. Finally, we arrange in descending order of delay and look at the five worst delays. That shows us the worst day, by a substantial margin, was [March 8](http://en.wikipedia.org/wiki/March_2013_nor%27easter). This code does the same thing as the first snippet, but it’s much easier to read.

`%>%` has analogues in many other languages: it’s similar to the pipe operator in F#, method chaining in JS and python, clojure’s thread-first macro, and Haskell’s `->`. It’s certainly not a new idea, but the implementation in R is novel, and it’s a very powerful idea. It’s really transformed the way that I code new data analyses.

It’s easy to write functions that work in a pipeline. A function only needs to adhere to a simple contract: the first argument should be the same type of thing as the last argument. This makes it easy to write pipelines for different types of object. You’ll learn more about different types of pipelines during [R day](http://strataconf.com/stratany2014/public/schedule/detail/37037) at [Strata + Hadoop World](http://strataconf.com/stratany2014) in October, including:

*   [tidyr](http://blog.rstudio.org/2014/07/22/introducing-tidyr/), pipelines to convert messy data into tidy (normalised) data that’s easy to wrangle, visualise and model.
*   [dplyr](http://blog.rstudio.org/2014/01/17/introducing-dplyr/), pipelines for data wrangling that work not just with in-memory data, but also with remote data stores, like databases and Google’s BigQuery.
*   [ggvis](http://blog.rstudio.org/2014/06/23/introducing-ggvis/), pipelines that let you create complex, powerful visualisations by starting simple and building your way up.

Data analysis doesn’t occur in a vacuum, so it’s crucial to be able get your work out of R and into the hands of decision makers. For that reason, we’ll also show you how to intermingle code and text into static reports with [rmarkdown](http://rmarkdown.rstudio.com/), and how to create engaging analytic experiences with [shiny](http://shiny.rstudio.com/). You’ll also learn about [packrat](http://rstudio.github.io/packrat/). Have you ever upgraded a package so you could use the latest and greatest feature, and then cursed when it broke old code in another project? Packrat solves that problem by managing an independent set of packages for each project.
