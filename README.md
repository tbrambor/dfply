# dfply

The dfply package makes it possible to do R's dplyr-style data manipulation with pipes
in python on pandas DataFrames.

This package is an alternative to [pandas-ply](https://github.com/coursera/pandas-ply)
and [dplython](https://github.com/dodger487/dplython) but heavily inspired by
both of them. In fact, the symbolic
representation of pandas DataFrames and Series objects for delayed calculation
is imported from pandas-ply.

The syntax and functionality of the package should in most cases look identical
to dplython. The major differences are in the code; dfply makes heavy use of
decorators to "categorize" the operation of data manipulation functions. The
goal of this choice of architecture is to make dfply concise and easily
extensible. There is a more in-depth overview of the decorators and how dfply can be
customized later in the readme.

<!-- START doctoc -->
<!-- END doctoc -->

## Overview

> (A notebook showcasing most of the working functions in dfply [can be
found here](https://github.com/kieferk/dfply/blob/master/examples/dfply-example-gallery.ipynb))

dfply works directly on pandas DataFrames, chaining operations on the data with
the `>>` operator.

```python
from dfply import *

diamonds >> head(3)

   carat      cut color clarity  depth  table  price     x     y     z
0   0.23    Ideal     E     SI2   61.5   55.0    326  3.95  3.98  2.43
1   0.21  Premium     E     SI1   59.8   61.0    326  3.89  3.84  2.31
2   0.23     Good     E     VS1   56.9   65.0    327  4.05  4.07  2.31
```

You can chain piped operations, and of course assign the output to a new
DataFrame.

```python
lowprice = diamonds >> head(10) >> tail(3)

lowprice

   carat        cut color clarity  depth  table  price     x     y     z
7   0.26  Very Good     H     SI1   61.9   55.0    337  4.07  4.11  2.53
8   0.22       Fair     E     VS2   65.1   61.0    337  3.87  3.78  2.49
9   0.23  Very Good     H     VS1   59.4   61.0    338  4.00  4.05  2.39
```

### Selecting and dropping

A variety of selection functions are available. The current functions are:

- `select(*columns)` returns the columns specified by the arguments
- `select_containing(*substrings)` returns columns containing substrings
- `select_startswith(*substrings)` returns columns starting with substrings
- `select_endswith(*substrings)` returns columns ending with substrings
- `select_between(column1, column2)` returns columns between two specified columns (inclusive)
- `select_to(column)` returns columns up to a specified column (exclusive)
- `select_from(column)` returns columns starting from a specified column (inclusive)
- `select_through(column)` returns columns up through a specified column (exclusive)

There are complimentary dropping functions to all the selection functions,
which will remove the specified columns instead of selecting them:

- `drop(*columns)`
- `drop_containing(*substrings)`
- `drop_startswith(*substrings)`
- `drop_endswith(*substrings)`
- `drop_between(column1, column2)`
- `drop_to(column)`
- `drop_from(column)`
- `drop_through(column)`

Column selection and dropping functions are designed to work with an arbitrary
combination of string labels, positional integers, or symbolic (`X.column`)
pandas Series objects.

The functions also "flatten" their arguments so that lists or tuples of selectors
will become individual arguments. This doesn't impact the user except for the
fact that you can mix single selectors and lists of selectors as arguments.

```python
diamonds >> select(1, X.price, ['x', 'y']) >> head(2)

       cut  price     x     y
0    Ideal    326  3.95  3.98
1  Premium    326  3.89  3.84
```

```python
diamonds >> drop_endswith('e','y','z') >> head(2)

   carat      cut color  depth     x
0   0.23    Ideal     E   61.5  3.95
1   0.21  Premium     E   59.8  3.89
```

```python
diamonds >> drop_through(X.clarity) >> select_to(X.x) >> head(2)

   depth  table  price
0   61.5   55.0    326
1   59.8   61.0    326
```

### Subsetting and filtering

Slices of rows can be selected with the `row_slice()` function. You can pass
single integer indices and/or lists of integer indices to select rows as with
pandas' `.iloc`.

```python
diamonds >> row_slice(1,2,[10,15])

    carat      cut color clarity  depth  table  price     x     y     z
1    0.21  Premium     E     SI1   59.8   61.0    326  3.89  3.84  2.31
2    0.23     Good     E     VS1   56.9   65.0    327  4.05  4.07  2.31
10   0.30     Good     J     SI1   64.0   55.0    339  4.25  4.28  2.73
15   0.32  Premium     E      I1   60.9   58.0    345  4.38  4.42  2.68
```

The `sample()` function functions exactly the same as pandas' `.sample()` method
for DataFrames. Arguments and keyword arguments will be passed through to the
DataFrame sample method.

```python
diamonds >> sample(frac=0.0001, replace=False)

       carat        cut color clarity  depth  table  price     x     y     z
19736   1.02      Ideal     E     VS1   62.2   54.0   8303  6.43  6.46  4.01
37159   0.32    Premium     D     VS2   60.3   60.0    972  4.44  4.42  2.67
1699    0.72  Very Good     E     VS2   63.8   57.0   3035  5.66  5.69  3.62
20955   1.71  Very Good     J     VS2   62.6   55.0   9170  7.58  7.65  4.77
5168    0.91  Very Good     E     SI2   63.0   56.0   3772  6.12  6.16  3.87


diamonds >> sample(n=3, replace=True)

       carat        cut color clarity  depth  table  price     x     y     z
52892   0.73  Very Good     G     SI1   60.6   59.0   2585  5.83  5.85  3.54
39454   0.57      Ideal     H     SI2   62.3   56.0   1077  5.31  5.28  3.30
39751   0.43      Ideal     H    VVS1   62.3   54.0   1094  4.84  4.85  3.02
```

Selection of unique rows is done with `distinct()`, which similarly passes
arguments and keyword arguments through to the DataFrame's `.drop_duplicates()`
method.

```python
diamonds >> distinct(X.color)

    carat        cut color clarity  depth  table  price     x     y     z
0    0.23      Ideal     E     SI2   61.5   55.0    326  3.95  3.98  2.43
3    0.29    Premium     I     VS2   62.4   58.0    334  4.20  4.23  2.63
4    0.31       Good     J     SI2   63.3   58.0    335  4.34  4.35  2.75
7    0.26  Very Good     H     SI1   61.9   55.0    337  4.07  4.11  2.53
12   0.22    Premium     F     SI1   60.4   61.0    342  3.88  3.84  2.33
25   0.23  Very Good     G    VVS2   60.4   58.0    354  3.97  4.01  2.41
28   0.23  Very Good     D     VS2   60.5   61.0    357  3.96  3.97  2.40
```

Filtering rows with logical criteria is done with `mask()`, which accepts
boolean arrays "masking out" False labeled rows and keeping True labeled rows.
These are best created with logical statements on symbolic Series objects as
shown below. Multiple criteria can be supplied as arguments and their intersection
will be used as the mask.

```python
diamonds >> mask(X.cut == 'Ideal') >> head(4)

    carat    cut color clarity  depth  table  price     x     y     z
0    0.23  Ideal     E     SI2   61.5   55.0    326  3.95  3.98  2.43
11   0.23  Ideal     J     VS1   62.8   56.0    340  3.93  3.90  2.46
13   0.31  Ideal     J     SI2   62.2   54.0    344  4.35  4.37  2.71
16   0.30  Ideal     I     SI2   62.0   54.0    348  4.31  4.34  2.68

diamonds >> mask(X.cut == 'Ideal', X.color == 'E', X.table < 55, X.price < 500)

       carat    cut color clarity  depth  table  price     x     y     z
26683   0.33  Ideal     E     SI2   62.2   54.0    427  4.44  4.46  2.77
32297   0.34  Ideal     E     SI2   62.4   54.0    454  4.49  4.52  2.81
40928   0.30  Ideal     E     SI1   61.6   54.0    499  4.32  4.35  2.67
50623   0.30  Ideal     E     SI2   62.1   54.0    401  4.32  4.35  2.69
50625   0.30  Ideal     E     SI2   62.0   54.0    401  4.33  4.35  2.69
```


### DataFrame transformation

New variables can be created with the `mutate()` function (named that to match
dplyr).

```python
diamonds >> mutate(x_plus_y=X.x + X.y) >> select_from('x') >> head(3)

      x     y     z  x_plus_y
0  3.95  3.98  2.43      7.93
1  3.89  3.84  2.31      7.73
2  4.05  4.07  2.31      8.12
```

Multiple variables can be created in a single call.

```python
diamonds >> mutate(x_plus_y=X.x + X.y, y_div_z=(X.y / X.z)) >> select_from('x') >> head(3)

      x     y     z   y_div_z  x_plus_y
0  3.95  3.98  2.43  1.637860      7.93
1  3.89  3.84  2.31  1.662338      7.73
2  4.05  4.07  2.31  1.761905      8.12
```

NOTE: because of python's unordered keyword arguments, the new variables
created with mutate are not (yet) guaranteed to be created in the same order
that they are input into the function call. This is on the todo list.

The `transmute()` function is a combination of a mutate and a selection of the
created variables.

```python
diamonds >> transmute(x_plus_y=X.x + X.y, y_div_z=(X.y / X.z)) >> head(3)

    y_div_z  x_plus_y
0  1.637860      7.93
1  1.662338      7.73
2  1.761905      8.12
```

### Grouping

DataFrames are grouped along variables using the `groupby()` function and
ungrouped with the `ungroup()` function. Functions chained after grouping a
DataFrame are applied by group until returning or ungrouping. Hierarchical/multiindexing is automatically removed.

In the example below, the `lead()` and `lag()` functions are dfply convenience
wrappers around the pandas `.shift()` Series method.


```python
(diamonds >> groupby(X.cut) >>
 mutate(price_lead=lead(X.price), price_lag=lag(X.price)) >>
 head(2) >> select(X.cut, X.price, X.price_lead))

          cut  price  price_lead  price_lag
8        Fair    337         NaN     2757.0
91       Fair   2757       337.0     2759.0
2        Good    327         NaN      335.0
4        Good    335       327.0      339.0
0       Ideal    326         NaN      340.0
11      Ideal    340       326.0      344.0
1     Premium    326         NaN      334.0
3     Premium    334       326.0      342.0
5   Very Good    336         NaN      336.0
6   Very Good    336       336.0      337.0
```


### Reshaping

Sorting is done by the `arrange()` function, which wraps around the pandas
`.sort_values()` DataFrame method. Arguments and keyword arguments are passed
through to that function (arguments are also currently flattened like in the select
functions).

```python
diamonds >> arrange(X.table, ascending=False) >> head(5)

       carat   cut color clarity  depth  table  price     x     y     z
24932   2.01  Fair     F     SI1   58.6   95.0  13387  8.32  8.31  4.87
50773   0.81  Fair     F     SI2   68.8   79.0   2301  5.26  5.20  3.58
51342   0.79  Fair     G     SI1   65.3   76.0   2362  5.52  5.13  3.35
52860   0.50  Fair     E     VS2   79.0   73.0   2579  5.21  5.18  4.09
49375   0.70  Fair     H     VS1   62.0   73.0   2100  5.65  5.54  3.47

(diamonds >> groupby(X.cut) >> arrange(X.price) >>
 head(3) >> ungroup() >> mask(X.carat < 0.23))
```


**...UNDER CONSTRUCTION...**




## TODO:

1. Variables created by mutate and transform are inserted into the
DataFrame in the same order that they appear in the function call.
- A variable created first in the mutate function can be used to create a
subsequent variable in the same function call.
- Not all functions have unit tests yet.
- More complete/advanced unit tests.
- Complete and improve function documentation.
- Better handling of numpy row indexers in row_slice. Should boolean be allowed
here? What about mixed boolean and integer? Currently not handled.
