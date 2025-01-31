Functional Programming
========================================================
author: Alejandro Schuler,  based on R for Data Science by Hadley Wickham
date: July 2021
transition: none
width: 1680
height: 1050



- write and test your own functions
- iterate functions over lists of arguments

Writing functions
============================================================
type: section

Motivation
===
- It's handy to be able to reuse your code and automate repetitive tasks
- Writing your own functions allows you to do that 
- When you write your code as functions, you can
  - name the function something evocative and readable
  - update the code in a single place instead of many
  - reduce the chance of making mistakes while copy-pasting
  - make your code shorter overall
  
Example
===
What does this code do? (note that `df$col` is a shortcut for `df %>% pull(col)`)

```r
df = tibble(
  a = rnorm(10), # 10 random numbers 
  b = rnorm(10),
  c = rnorm(10),
  d = rnorm(10)
)
```

```r
df$a = (df$a - min(df$a, na.rm = TRUE)) / 
  (max(df$a, na.rm = TRUE) - min(df$a, na.rm = TRUE))
df$b = (df$b - min(df$b, na.rm = TRUE)) / 
  (max(df$b, na.rm = TRUE) - min(df$a, na.rm = TRUE))
df$c = (df$c - min(df$c, na.rm = TRUE)) / 
  (max(df$c, na.rm = TRUE) - min(df$c, na.rm = TRUE))
df$d = (df$d - min(df$d, na.rm = TRUE)) / 
  (max(df$d, na.rm = TRUE) - min(df$d, na.rm = TRUE))
```

Example
===
(recall that `df$col` is the same as `df %>% pull(col)`)

```r
df$a = (df$a - min(df$a, na.rm = TRUE)) / 
  (max(df$a, na.rm = TRUE) - min(df$a, na.rm = TRUE))
df$b = (df$b - min(df$b, na.rm = TRUE)) / 
  (max(df$b, na.rm = TRUE) - min(df$a, na.rm = TRUE))
df$c = (df$c - min(df$c, na.rm = TRUE)) / 
  (max(df$c, na.rm = TRUE) - min(df$c, na.rm = TRUE))
df$d = (df$d - min(df$d, na.rm = TRUE)) / 
  (max(df$d, na.rm = TRUE) - min(df$d, na.rm = TRUE))
```
- It looks like we're standardizing all the variables by their range so that they fall between 0 and 1
- But did you spot the mistake? The code runs with no errors...

Example
===

```r
rescale = function(vec) {
  (vec - min(vec))/(max(vec) - min(vec))
}

df2 = df %>%
  mutate(
    a= rescale(a),
    b= rescale(b),
    c= rescale(c),
    d= rescale(d),
  )
```
- Much improved!
- The last two lines clearly say: replace all the columns with their rescaled versions
  - This is because the function name `rescale()` is informative and communicates what it does
  - If a user (or you a few weeks later) is curious about the specifics, they can check the function body

Example
===

```r
rescale = function(vec) {
  (vec - min(vec))/(max(vec) - min(vec))
}

df2 = df %>%
  mutate(
    across(a:d, rescale)
  ) # see ?across
```
- Even better.
- ... now we notice that `min()` is being computed twice in the function body, which is inefficient
- We are also not accounting for NAs

Example
===

```r
rescale = function(vec) {
  vec_rng = range(vec, na.rm=T) # same as c(min(vec,na.rm=T), max(vec,na.rm=T))
  (vec - vec_rng[1])/(vec_rng[2] - vec_rng[1])
}

df2 = df %>%
  mutate(across(a:d, rescale))
```
- Since we have a function, we can make the change in a single place and improve the efficiency of multiple parts of our code
- Bonus question: why use `range()` instead of getting and saving the results of `min()` and `max()` separately?

Example
===
We can also test our function in cases where we know what the output should be to make sure it works as intended before we let it loose on the real data

```r
rescale(c(0,0,0,0,0,1))
[1] 0 0 0 0 0 1
rescale(0:10)
 [1] 0.0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 1.0
rescale(-10:0)
 [1] 0.0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 1.0
x = c(0,1,runif(100))
all(x == rescale(x))
[1] TRUE
```
- These tests are a critical part of writing good code! It is helpful to save your tests in a separate file and organize them as you go

Function declaration syntax
===
To write a function, just wrap your code in some special syntax that tells it what variables will be passed in and what will be returned

```r
rescale = function(x) {
  x_rng = range(x, na.rm=T) 
  (x - x_rng[1])/(x_rng[2] - x_rng[1])
}
```

- Just like assigning a variable, except what you put into `FUNCTION_NAME` now isn't a data frame, vector, etc, it's a function object that gets created by the `function(..) {...}` syntax
- At any point in the body you can `return()` the value, or R will automatically return the result of the last line of code in the body that gets run
- once declared, it can be called:

```r
rescale(0:10)
 [1] 0.0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 1.0
```


***

- The syntax is `FUNCTION_NAME <- function(ARGUMENTS...) { CODE }`
- what you call the arguments that go in the `function(...)` part is how the function will refer to these inputs internally and specify how it should be called using named arguments

![](https://github.com/alejandroschuler/r4ds-courses/blob/summer-2023/figures/call.png?raw=true)

```
aes = function(x,y) {...} # defining 
aes(x=EIF3L, y=VAPA) # calling 
```


Optional arguments
===
To add an **optional** argument, add an `=` after you declare it as a variable and write in the default value that you would like that variable to take

```r
rescale = function(x, na.rm=TRUE) {
  x_rng = range(x, na.rm=na.rm) 
  (x - x_rng[1])/(x_rng[2] - x_rng[1])
}

vec = c(0,1,NA)

rescale(vec)
[1]  0  1 NA
rescale(vec, na.rm=T)
[1]  0  1 NA
rescale(vec, na.rm=F)
[1] NA NA NA
```
- All optional arguments must go after mandatory arguments in the function declaration

Exercise: Reverse
===
type:prompt

- write a function that takes a single vector or list as input and returns it in reverse order

Exercise: Quadratic equation
===
type:prompt
- write a function that returns the larger of the two roots of a polynomial Ax^2 + Bx + C obtained using the quadratic equation. 
- your function should take as input three arguments: `A`, `B`, and `C`, and should return a single number (possibly `NaN` if there turn out to be no roots)
- test your function with a variety of inputs by checking that the returned solution x does satisfy Ax^2 + Bx + C = 0.

Exercise: Hardcoding na.rm
===
type:prompt
- It's annoying that the `sum()` function returns `NA` if any values of the input vector are `NA`. You can fix this by passing in the optional argument `na.rm=T` every time you call `sum()`, but it's inconvenient to type that every single time.
- Using a single line of code, write a new function (called `sum_obs()`, short for "sum observed") that takes a vector and returns the sum of all the non-NA values. Your function should call the usual `sum()` internally.

Exercise: NAs in two vectors
===
type: prompt
- Write a function called both_na() that takes two vectors of the same length and returns the total number of positions that have an NA in both vectors
- Make a few vectors and test out your code

Functions returning multiple values
===
- A function can only return a single object
- Often, however, it makes sense to group the calculation of two or more things you want to return within a single function
- You can put all of that into a list and then return a single list

```r
min_max = function(x) {
  x_sorted = sort(x)
  list(
    min = x_sorted[1],
    max = x_sorted[length(x)]
  )
}
```
- Why might this code be preferable to running `min()` and then `max()`?

***

- A `list` is like a vector, except the elements don't have to be the same type of thing

```r
c(1,2,"hello",5,TRUE)
[1] "1"     "2"     "hello" "5"     "TRUE" 

list(1,2,"hello",5,TRUE)
[[1]]
[1] 1

[[2]]
[1] 2

[[3]]
[1] "hello"

[[4]]
[1] 5

[[5]]
[1] TRUE
```

Functions are objects
===

```r
rescale
function(x, na.rm=TRUE) {
  x_rng = range(x, na.rm=na.rm) 
  (x - x_rng[1])/(x_rng[2] - x_rng[1])
}
<bytecode: 0x10a7df138>
```
- Because of this, they themselves can be passed as arguments to other functions

```r
df2 = df %>% mutate(across(a:d, rescale))
```
- This is what **functional programming** means. The functions themselves are can be treated as regular objects like variables
- The name of the function is just what you call the "box" that the function (the code) lives in, just like variables names are names for "boxes" that contain data

Exercise: Quadratic equation
===
type:prompt
- modify your quadratic equation function so that it returns **both** roots of a polynomial Ax^2 + Bx + C. 

Iteration
===
type:section

Map
===
- Map is a function that takes a list (or vector) as its first argument and a function as its second argument
- Recall that functions are objects just like anything else so you can pass them around to other functions
- Map then runs that function on each element of the first argument, slaps 
the results together into a list, and returns that

```r
grades = list(
  class_A = c(90, 87, 92, 78, 69),
  class_B = c(88, 85, 76, 78, 77, 97, 91)
)
map(grades, mean)
$class_A
[1] 83.2

$class_B
[1] 84.57143
```
- map preserves list names

***

Or:

```r
grades %>%
  map(mean)
$class_A
[1] 83.2

$class_B
[1] 84.57143
```


```r
grades %>%
  map(max)
$class_A
[1] 92

$class_B
[1] 97
```


```r
grades %>%
  map(min)
$class_A
[1] 69

$class_B
[1] 76
```

Map 
===
- as another example, let's say you want to read in multiple files:

```r
url_start = "https://raw.githubusercontent.com/alejandroschuler/r4ds-courses/summer-2023/data/gtex_metadata/"
files = list(
  samples = "gtex_samples_time.csv",
  tissues = "gtex_tissue_month_year.csv",
  dates = "gtex_dates_clean.csv"
)

urls = str_c(url_start, files)
```


```r
data_frames = urls %>%
  map(read_csv)
```


Anonymous function syntax
===
- up until now we had to define our functions outside of map and then pass them in as an argument:

```r
count_rc = function(df) {
  tibble(
    n_rows = nrow(df),
    n_cols = ncol(df)
  )
}

data_frames %>%
  map_df(count_rc)
```

***
- Instead, we can define a function inside of another function call.
- These functions are "anonymous" because they are never assigned a name and will not be used again


```r
data_frames %>%
  map_df(\(df) tibble(
    n_rows = nrow(df),
    n_cols = ncol(df)
  ))
```

- the syntax is `\(ARGUMENTS) BODY`
- just an abbreviation for `function(ARGUMENTS) {BODY}`

Map or data frame?
===

- in our initial `map()` example we were looking at some course grades:

```r
grades
$class_A
[1] 90 87 92 78 69

$class_B
[1] 88 85 76 78 77 97 91

grades %>% map(mean)
$class_A
[1] 83.2

$class_B
[1] 84.57143

grades %>% map(max)
$class_A
[1] 92

$class_B
[1] 97

grades %>% map(min)
$class_A
[1] 69

$class_B
[1] 76
```

***

- these operations could be more organized if we made a data frame:

```r
grades_df = grades %>%
  imap(\(vec, name) 
    tibble(class=name, grade=vec)
  ) %>%
  bind_rows

grades_df %>%
  group_by(class) %>%
  summarize(
    mean(grade), 
    max(grade), 
    min(grade)
  )
# A tibble: 2 × 4
  class   `mean(grade)` `max(grade)` `min(grade)`
  <chr>           <dbl>        <dbl>        <dbl>
1 class_A          83.2           92           69
2 class_B          84.6           97           76
```
- we'll learn about `imap` shortly

Exercise: map practice
===
type: prompt


```r
url_start = "https://raw.githubusercontent.com/alejandroschuler/r4ds-courses/summer-2023/data/gtex_metadata/"

data_frames = list(
  samples = "gtex_samples_time.csv",
  tissues = "gtex_tissue_month_year.csv",
  dates = "gtex_dates_clean.csv"
)  %>% 
  map(\(f) str_c(url_start, f)) %>%
  map(read_csv)
```

`data_frames` is a list of three data frames. Use map to output:

1. the number of rows of each table
2. the number of columns of each table

Exercise: read files
===
type: prompt

Earlier we saw this example of reading in multiple files:

```r
url_start = "https://raw.githubusercontent.com/alejandroschuler/r4ds-courses/summer-2023/data/gtex_metadata/"

data_frames = list(
  samples = "gtex_samples_time.csv",
  tissues = "gtex_tissue_month_year.csv",
  dates = "gtex_dates_clean.csv"
)  %>% 
  map(\(f) str_c(url_start, f)) %>%
  map(read_csv)
```

- modify the code so that only the first 10 lines of each file are read in.

Exercise: map practice
===
type: prompt
- For each value of x in [1, 5, 10] Generate 10 random numbers between 0 and x (see: `?runif`).

Returning other data types
===
- `map()` typically returns a list (why?)
- But there are variants that return different data types


```r
data_frames %>%
  map_dbl(nrow)
samples tissues   dates 
     66    1475    1234 
```


```r
count_rc = function(df) {
  tibble(
    n_rows = nrow(df),
    n_cols = ncol(df)
  )
}

data_frames %>%
  map_df(count_rc)
# A tibble: 3 × 2
  n_rows n_cols
   <int>  <int>
1     66      3
2   1475      4
3   1234      6
```

Mapping over multiple inputs
===
- So far we’ve mapped along a single input. But often you have multiple related inputs that you need iterate along in parallel. That’s the job of `pmap()`. For example, imagine you want to draw a random numbers between `a` and `b` as both of those vary:

```r
a = c(1,2,3)
b = c(2,3,4)

runif(1, a[1], b[1])
[1] 1.29373
runif(1, a[2], b[2])
[1] 2.19126
runif(1, a[3], b[3])
[1] 3.886451
```

***
- `pmap` makes this easier:

```r
list(
  a = c(1,2,3),
  b = c(2,3,4)
) %>% pmap(
  \(a,b) runif(1,a,b)
)
[[1]]
[1] 1.503339

[[2]]
[1] 2.877058

[[3]]
[1] 3.189194
```


<div align="center">
<img src="https://dcl-prog.stanford.edu/images/pmap-flipped.png">
</div>

Mapping over names
===
- `imap()` lets you operate on the **names** of the input list.

```r
grades
$class_A
[1] 90 87 92 78 69

$class_B
[1] 88 85 76 78 77 97 91
```

*** 


```r
grades %>% imap(
  \(value, name) 
  tibble(grade=value, class=name)
)
$class_A
# A tibble: 5 × 2
  grade class  
  <dbl> <chr>  
1    90 class_A
2    87 class_A
3    92 class_A
4    78 class_A
5    69 class_A

$class_B
# A tibble: 7 × 2
  grade class  
  <dbl> <chr>  
1    88 class_B
2    85 class_B
3    76 class_B
4    78 class_B
5    77 class_B
6    97 class_B
7    91 class_B
```

Creating a grid of values
=== 
- `expand_grid()` gives you every combination of the items in the list you pass it

```r
expand_grid(
    a = c(1,2,3),
    b = c(10,11)
  )
# A tibble: 6 × 2
      a     b
  <dbl> <dbl>
1     1    10
2     1    11
3     2    10
4     2    11
5     3    10
6     3    11
```

Exercise: dimensions
===
type:prompt


```r
url_start = "https://raw.githubusercontent.com/alejandroschuler/r4ds-courses/summer-2023/data/gtex_metadata/"

data_frames = list(
  samples = "gtex_samples_time.csv",
  tissues = "gtex_tissue_month_year.csv",
  dates = "gtex_dates_clean.csv"
)  %>% 
  map(\(f) str_c(url_start, f)) %>%
  map(read_csv)
```

Fill in the missing parts of the code below to programmatically create the following table:



```r
data_frames %>%
  imap(
    ???
  ) %>%
  bind_rows()
```


```
# A tibble: 3 × 3
  dataset  nrow  ncol
  <chr>   <int> <int>
1 samples    66     3
2 tissues  1475     4
3 dates    1234     6
```


Exercise: reading files in multiple directories 
===
type: prompt
My collaborator has an online folder of experimental results named `results` that can be found at `"https://raw.githubusercontent.com/alejandroschuler/r4ds-courses/summer-2023/data/results"`. In that folder, there are 20 sub-folders that represent the results of each repetition of her experiment. These sub-folders are each named `rep_n`, so, e.g. `results/rep_14` would be one sub-folder. Within each sub-folder, there are 3 csv files called `a.csv`, `b.csv` `c.csv` that contain different kinds of measurements. Thus, a full path to one of these files might be `results/rep_14/c.csv`.

1. write code to read these all into one long list of data frames. `str_c()` or `glue()` will be helpful to create the required file names


2. Unfortunately, that wasn't helpful because now you don't know what data frames are what results. Consider just the "`a`" files. Write code that reads in only the "`a`" files and concatenates them into one data frame. Include a column in this data frame that indicates which experimental repetition each row of the data frame came from (use `imap()`).


3. Turn your code from above into a function that takes as input the file name (`'a'`, for example) and returns the single concatenated file. Iterate that function over the different file names to output three master data frames corresponding to the file types `'a'`, `'b'`, and `'c'`.


Why not for loops?
===
- R also provides something called a `for` loop, which is common to many other languages as well. It looks like this:

```r
data_frames = list(NA, NA, NA)
for (i in 1:3) {
  data_frames[[i]] = read_csv(urls[[i]])
}
```
- The `for` loop is very flexible and you can do a lot with it
- `for` loops are unavoidable when the result of one iteration depends on the result of the last iteration

***

- Compare to the `map()`-style solution:

```r
data_frames = urls %>%
  map(read_csv)
```
- Compared to the `for` loop, the `map()` syntax is much more concise and eliminates much of the "admin" code in the loop (setting up indices, initializing the list that will be filled in, indexing into the data structures)
- The `map()` syntax also encourages you to write a function for whatever is happening inside the loop. This means you have something that's reusable and easily testable, and your code will look cleaner
- Loops in R can be catastrophically slow due to the complexities of [copy-on-modify semantics](https://adv-r.hadley.nz/names-values.html). 

purrr Cheatsheet
===
<div align="center">
<img src="https://rstudio.com/wp-content/uploads/2018/08/purrr.png"
</div>

