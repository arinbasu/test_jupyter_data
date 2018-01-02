## A Primer on how to conduct time series and survival analysis in public health using Jupyter Notebook

In this module, we will learn how to conduct time series and survival analysis in R using a Jupyter notebook and publish our results through github or through other tools. In general, we will follow the same principles true for any kind of data analysis:

1. First, you will need to "clean" the data set so that this is ready for your use.
2. Second, you will need to explore the data set so that you understand the nature of the variables that you will be working with
3. Third, you either work with the entire data set or create 


```R
## load the needed packages
library(tidyr)
library(knitr)
library(survival)
library(tidyverse)
```

    ── Attaching packages ─────────────────────────────────────── tidyverse 1.2.1 ──
    ✔ tibble  1.3.4     ✔ dplyr   0.7.4
    ✔ readr   1.1.1     ✔ stringr 1.2.0
    ✔ purrr   0.2.4     ✔ forcats 0.2.0
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ magrittr::extract() masks tidyr::extract()
    ✖ dplyr::filter()     masks stats::filter()
    ✖ dplyr::lag()        masks stats::lag()
    ✖ purrr::set_names()  masks magrittr::set_names()



```R
library(survminer)
```


```R
## we will read a 100 person data set for this 

mydata <- read_table("second_edition_data/whas100.dat",
                    col_names = F)

names(mydata) = c("id", "admission_date", "follow_up_date",
                "length_stay", "follow_up_time",
                "vital_status", "age_admission",
                "gender", "bmi")
# the admission_date and follow-up_date needs to be formatted into character

library(lubridate)
mydata$admission_date <- mdy(mydata$admission_date)
mydata$follow_up_date <- mdy(mydata$follow_up_date)

str(mydata)
head(mydata, n = 6L)
```

    Parsed with column specification:
    cols(
      X1 = col_integer(),
      X2 = col_character(),
      X3 = col_character(),
      X4 = col_integer(),
      X5 = col_integer(),
      X6 = col_integer(),
      X7 = col_integer(),
      X8 = col_integer(),
      X9 = col_double()
    )


    Classes ‘tbl_df’, ‘tbl’ and 'data.frame':	100 obs. of  9 variables:
     $ id            : int  1 2 3 4 5 6 7 8 9 10 ...
     $ admission_date: Date, format: "1995-03-13" "1995-01-14" ...
     $ follow_up_date: Date, format: "1995-03-19" "1996-01-23" ...
     $ length_stay   : int  4 5 5 9 4 7 3 56 5 9 ...
     $ follow_up_time: int  6 374 2421 98 1205 2065 1002 2201 189 2719 ...
     $ vital_status  : int  1 1 1 1 1 1 1 1 1 0 ...
     $ age_admission : int  65 88 77 81 78 82 66 81 76 40 ...
     $ gender        : int  0 1 0 1 0 1 1 1 0 0 ...
     $ bmi           : num  31.4 22.7 27.9 21.5 30.7 ...
     - attr(*, "problems")=Classes ‘tbl_df’, ‘tbl’ and 'data.frame':	1 obs. of  4 variables:
      ..$ row     : int 101
      ..$ col     : int 1
      ..$ expected: chr "2 chars between fields"
      ..$ actual  : chr "0 chars until end of line"
     - attr(*, "spec")=List of 2
      ..$ cols   :List of 9
      .. ..$ X1: list()
      .. .. ..- attr(*, "class")= chr  "collector_integer" "collector"
      .. ..$ X2: list()
      .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
      .. ..$ X3: list()
      .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
      .. ..$ X4: list()
      .. .. ..- attr(*, "class")= chr  "collector_integer" "collector"
      .. ..$ X5: list()
      .. .. ..- attr(*, "class")= chr  "collector_integer" "collector"
      .. ..$ X6: list()
      .. .. ..- attr(*, "class")= chr  "collector_integer" "collector"
      .. ..$ X7: list()
      .. .. ..- attr(*, "class")= chr  "collector_integer" "collector"
      .. ..$ X8: list()
      .. .. ..- attr(*, "class")= chr  "collector_integer" "collector"
      .. ..$ X9: list()
      .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
      ..$ default: list()
      .. ..- attr(*, "class")= chr  "collector_guess" "collector"
      ..- attr(*, "class")= chr "col_spec"



<table>
<thead><tr><th scope=col>id</th><th scope=col>admission_date</th><th scope=col>follow_up_date</th><th scope=col>length_stay</th><th scope=col>follow_up_time</th><th scope=col>vital_status</th><th scope=col>age_admission</th><th scope=col>gender</th><th scope=col>bmi</th></tr></thead>
<tbody>
	<tr><td>1         </td><td>1995-03-13</td><td>1995-03-19</td><td>4         </td><td>   6      </td><td>1         </td><td>65        </td><td>0         </td><td>31.38134  </td></tr>
	<tr><td>2         </td><td>1995-01-14</td><td>1996-01-23</td><td>5         </td><td> 374      </td><td>1         </td><td>88        </td><td>1         </td><td>22.65790  </td></tr>
	<tr><td>3         </td><td>1995-02-17</td><td>2001-10-04</td><td>5         </td><td>2421      </td><td>1         </td><td>77        </td><td>0         </td><td>27.87892  </td></tr>
	<tr><td>4         </td><td>1995-04-07</td><td>1995-07-14</td><td>9         </td><td>  98      </td><td>1         </td><td>81        </td><td>1         </td><td>21.47878  </td></tr>
	<tr><td>5         </td><td>1995-02-09</td><td>1998-05-29</td><td>4         </td><td>1205      </td><td>1         </td><td>78        </td><td>0         </td><td>30.70601  </td></tr>
	<tr><td>6         </td><td>1995-01-16</td><td>2000-09-11</td><td>7         </td><td>2065      </td><td>1         </td><td>82        </td><td>1         </td><td>26.45294  </td></tr>
</tbody>
</table>




```R
?head
```



<table width="100%" summary="page for head {utils}"><tr><td>head {utils}</td><td style="text-align: right;">R Documentation</td></tr></table>

<h2>
Return the First or Last Part of an Object
</h2>

<h3>Description</h3>

<p>Returns the first or last parts of a vector, matrix, table, data frame
or function.  Since <code>head()</code> and <code>tail()</code> are generic
functions, they may also have been extended to other classes.
</p>


<h3>Usage</h3>

<pre>
head(x, ...)
## Default S3 method:
head(x, n = 6L, ...)
## S3 method for class 'data.frame'
head(x, n = 6L, ...)
## S3 method for class 'matrix'
head(x, n = 6L, ...)
## S3 method for class 'ftable'
head(x, n = 6L, ...)
## S3 method for class 'table'
head(x, n = 6L, ...)
## S3 method for class 'function'
head(x, n = 6L, ...)

tail(x, ...)
## Default S3 method:
tail(x, n = 6L, ...)
## S3 method for class 'data.frame'
tail(x, n = 6L, ...)
## S3 method for class 'matrix'
tail(x, n = 6L, addrownums = TRUE, ...)
## S3 method for class 'ftable'
tail(x, n = 6L, addrownums = FALSE, ...)
## S3 method for class 'table'
tail(x, n = 6L, addrownums = TRUE, ...)
## S3 method for class 'function'
tail(x, n = 6L, ...)
</pre>


<h3>Arguments</h3>

<table summary="R argblock">
<tr valign="top"><td><code>x</code></td>
<td>
<p>an object</p>
</td></tr>
<tr valign="top"><td><code>n</code></td>
<td>
<p>a single integer. If positive, size for the resulting
object: number of elements for a vector (including lists), rows for
a matrix or data frame or lines for a function. If negative, all but
the <code>n</code> last/first number of elements of <code>x</code>.</p>
</td></tr>
<tr valign="top"><td><code>addrownums</code></td>
<td>
<p>if there are no row names, create them from the row
numbers.</p>
</td></tr>
<tr valign="top"><td><code>...</code></td>
<td>
<p>arguments to be passed to or from other methods.</p>
</td></tr>
</table>


<h3>Details</h3>

<p>For matrices, 2-dim tables and data frames, <code>head()</code> (<code>tail()</code>) returns
the first (last) <code>n</code> rows when <code>n &gt; 0</code> or all but the
last (first) <code>n</code> rows when <code>n &lt; 0</code>.  <code>head.matrix()</code> and
<code>tail.matrix()</code> are exported.  For functions, the
lines of the deparsed function are returned as character strings.
</p>
<p>If a matrix has no row names, then <code>tail()</code> will add row names of
the form <code>"[n,]"</code> to the result, so that it looks similar to the
last lines of <code>x</code> when printed.  Setting <code>addrownums =
    FALSE</code> suppresses this behaviour.
</p>


<h3>Value</h3>

<p>An object (usually) like <code>x</code> but generally smaller.  For
<code>ftable</code> objects <code>x</code>, a transformed <code>format(x)</code>.
</p>


<h3>Author(s)</h3>

<p>Patrick Burns, improved and corrected by R-Core. Negative argument
added by Vincent Goulet.
</p>


<h3>Examples</h3>

<pre>
head(letters)
head(letters, n = -6L)

head(freeny.x, n = 10L)
head(freeny.y)

tail(letters)
tail(letters, n = -6L)

tail(freeny.x)
tail(freeny.y)

tail(library)

head(stats::ftable(Titanic))
</pre>

<hr /><div style="text-align: center;">[Package <em>utils</em> version 3.4.3 ]</div>

