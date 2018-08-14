R notes 
===============

[Advanced R](http://adv-r.had.co.nz/) (this notes cover up to the function section, then performance parts)

## Basics

#### Basic Types 

```R
x <- 3 		# value assignment
		    # x = 3 also works, but usually when setting function arguments 
17 %% 3		# mod, returns 2
17 %/% 3	# integer division, returns 5

?log 		# get help on anything
class(x)	# show class name 
 			# 	for integer class, use x <- 123L
paste(a,b,sep='')	# join strings. When a or b are vectors, the result is the same as the longer one, while the shorter one is used in periods 

# type conversion 
as.integer(x)	# convert to integer

# logical: TRUE and FALSE are reserved keywords, T and F are defined as T <- TRUE and F <- FALSE
```

There are pre-defined variables for convenience (e.g. `T`, `F`, `pi`). They are defined in R's `base` package and can be overwritten (as variable search goes to `base` package last). 

**Note**: R allows using dot in names. `as.integer` is actually the name of a function. 

**Note** `nchar` is used for finding length of a string (class `character`). `length` is used for finding length of string *vector* and taking `length` of a normal string will always give 1. 

##### NA, NaN, Inf and NULL

R has special types `NA`, `NaN` and `NULL`. 

- `NA` (not applicable) is available in all classes. It can be checked with `is.na`. It's not an `NaN`. 
- `NaN` (not a number) is only available in `numeric` class and can be checked with `is.nan`. It's still considered not applicable (`is.na(NaN)` gives `TRUE`)
- `Inf` (infinity) is only available in `numeric` class and can be checked with `is.infinite`. It's not considered NA or NaN. 
- `NULL` is a special class meaning nothing is there. It can only be checked with `is.null(NULL)` (or `length(NULL) == 0`)
  - **Note** the length of `NULL` is 0, while the length of `NA` and `NaN` is 1. 
  - **Note** there is no `NULL` vector. `c(NULL, NULL)` still gives `NULL`, as the length for `NULL` class instance is always 0.

**Note**: A good way to remove `NA`, `NaN` and `Inf` is using `is.finite`. Note that this doesn't work for `NULL`, which gives `logical(0)`. 

#### Vector

Vectors are 1-dimensional array of primitives with the same types. Actually primitive types are considered as vectors of length 1. 

```R
c(a,b)			# concatenate into a single vector
length(x)		# return length of the vector. Use length(x) > 1 to check for vector vs. primitive
class(c(1,2))	# returns 'numeric', NOT vector
x <- 1:5		# an integer vector from 1 to 5
sep(start, stop, interval)	# like start:interval:stop in MATLAB. sep(1,5,3) gives 1,4
rep(c(1,2),3)	# repeat. 1,2,1,2,1,2
rep(c(1,2),each=3)	# repeat each. 1,1,1,2,2,2
TYPENAME(n)		# preallocate a vector of length n. e.g. numeric(10)
sum(c(1,NA,2),na.rm=TRUE)	# many functions have na.rm argument to ignore all NA in the vector
```

**Note**: numeric functions all work on vectors element-wisely. When the two vectors involved are of different size, the longer one's size must be a multiple of shorter one's, and the shorter one will be *expanded automatically*. 

**Note**: assigning value to an element of the vector may lead to coercion of the entire vector: 

```R
x = 1:5
class(x) # Integer
x[1] = 1 # note that this is 1, not 1L
class(x) # Numeric
```

**Note**: `is.vector()` tests if a vector has attributes other than `names`. It does not test if an object is a vector. 

**Vector indexing**

- vectors can take vector of integers as index
- indices are 1-based
- index 0 is ignored
- Boolean vector can be used (`x[x < 0]` chooses all negative values)
- indices can have names, can elements can be accessed using name/index

```R
x[i]	# returns the i-th element
x[-i]	# NOTE: returns x without the i-th element
x[c(1,0,2)]		# same as x[c(1,2)], since 0 is ignored
which(x > 0)		# only works for logical vector
names(x) <- c("a", "b", "c")
x["a"]	# index by name or index are both accepted
```

**Note**: when indexing with logical vector, the repeating behavior is also used. 

```R
(1:6)[c(TRUE,FALSE)] # gives c(1,3,5)
(1:6)[c(TRUE,FALSE,NA)] # gives c(1,NA,4,NA)
```



**Set**

- Vectors can be used as sets
- `union`, `intersect`, `setdiff`, `setequal` are available for set operations
  - When a set is created, duplicates are discarded
  - The order of elements are determined by concatenate vectors, and then removing elements accordingly (for duplicates, keep the first one)

```R
is.element(a,b)		# for each element of a, return TRUE if it's in b
a %in% b			# same as is.element(a,b)
```

#### Matrix

- Use `%...%` for matrix operations, and without `%` for component-wise operations
- All columns and rows can have names. Like vectors, names and indices can both be used for indexing 
- Index matrix (size n*2) can be used for indexing
- **Note**: matrix elements are saved in memory in a column-first order (like MATLAB, unlike C). 
- **Note**: running `x[1]` without comma will access matrix elements as if viewing the underlying storage as a vector, so it always returns a number. 
- **Note** `cbind()` and `rbind()` also follows the repeating behavior that applies to numeric functions of r. e.g. `cbind(1,2:3)` gives `[1,2;1;3]`. 

```R
z <- matrix(x,nrow=m,ncol=n)	# create a matrix with m rows and n columns, filled with x
							  # when x is a vector, the underlying storage is filled by rep(x,m*n/length(x))
dim(z)			# shape of matrix (an integer vector)
length(z)		# number of elements in the matrix
class(z)		# matrix is a class, or rather a wrapper on vectors
as.numeric(z)	# view the underlying storage as a vector
z %*% t(z)		# %*% is used for matrix multiplication. * is for component-wise multiplication
cbind(z,z)		# combine as columns
rbind(z,z)		# combine as rows

z[a,]			# a-th row 
z[,b]			# b-th column
z[a,,drop=FALSE]	# use drop=FALSE to keep dimensions and return a matrix. Note that the default return type is a vector
rownames(z)		# set/get rownames of z. colnames is also available
```

#### data.frame

A data.frame contains columns of vectors of the same length, with indexing syntax same as matrix. 

`z$a`: used to select one column of data

- **Note**: partial name matching is used for `$`. `NULL` is returned when multiple matchings are available. 
- for assignment, full name must be used. partial name matching won't work for assignment.  
- Use `z$a<-NULL` to remove a column. 
- **Note**: unlike matrix, for dataframe `z[1]` (without comma) actually access columns (`z[1]` gives the first column), and not `z[1,]`. 
- **Note** matrix and list can be used as one column of dataframe. Look out for that. 
- Tip: `z[1000 + -10:10]` useful pattern to inspect data before/after

#### list

like the list in python, each element of the list can have a different type, and normal math functions in vector and matrix won't work. 

```r
l = list(3,1:5,4)
l = list(a=3, 1:5, 4)	# list element can also have names
l[1]	# return a list of 1 element (list slice)
l[[1]]	# return the element itself
l[["a"]] # index by name or index are both supported
l$a 	# index using $ is supported
l[c(1,2)]	# return a list composed of the first 2 elements of l

c(l,l)	# merge two lists
unlist(l)	# convert back to vector, with values 3,1,2,3,4,5,4. Note that the type is infered. 
```

**Note**: dataframes are actually lists with the extra restriction that all elements must have the same length. 

**Note**: `c()` works differently for vectors and lists

```R
c(1,c(c(2,3))) # c(1,2,3)
c(list(1), list(list(2,3))) # list(1,list(2,3))
```

- For vectors, `c()` works as concatenation (primitive types are vectors of size 1)
- For lists, `c()` works as concatenation and `list()` construct a list with everything
  - when combining lists with vectors, vectors will first be coerced into a list with one elements

**Note**: depending on syntax, R may try to simplify/preserve the data type/class and structure of input (e.g. names, unused levels).

|            | Simplifying              | Preserving                                   | Simplifying behavior                    |
| ---------- | ------------------------ | -------------------------------------------- | --------------------------------------- |
| Vector     | `x[[1]]`                 | `x[1]`                                       | remove names/attributes                 |
| List       | `x[[1]]`                 | `x[1]`                                       | return the item, not a slice of list    |
| Factor     | `x[1:4, drop = T]`       | `x[1:4]`                                     | drop unused levels                      |
| Array      | `x[1, ]` **or** `x[, 1]` | `x[1, , drop = F]` **or** `x[, 1, drop = F]` | drop singleton dimension                |
| Data frame | `x[, 1]` **or** `x[[1]]` | `x[, 1, drop = F]` **or** `x[1]`             | return a vector if it's a single column |



#### factor

Factor is a class used like enum. It's implemented as an integer vector with an attribute `levels`. 

- **Note**: when coerced into a numeric vector, the underlying integer vector is returned (i.e. the level of each element)
  - To get the vector of levels from a vector using integer for levels, use `as.integer(as.character(x))`
- Most data loading function in r turns strings into factors, including `read.csv()`. To avoid this behavior, use `stringAsFactors=FALSE`. 
- data frames also turns strings into factors by default. 

#### Attributes

All objects can have attributes. 

- `attr()` is used to set/get individual attributes
- `attributes()` is used to set/get all attributes
- An object with additional attributes can be constructed by `structure(obj, <attr_name>=<attr_value>)`
- Attributes are lost when modifying a vector. To keep it, use indexing by nothing (`v[]<-f(v)` instead of `v<-f(v)`)

#### Memory

- R uses value-copy, not reference (like in Python)

- R uses copy on modify, i.e. if `x <- y`, the memory only doubled when `x` or `y` is modified

- Due to the value-copy feature, reference to partial storage is unavailable

  ```R
  diag(z) <- c(1,2)		# set diagonal of z
  x <- diag(z)
  x <- c(1,2)				# doesn't work this way
  ```

```R
ls()		# list all variables
rm(x)		# remove variable x
rm(list=ls())	# remove all variables
gc()		# force a garbage colletion
gcinfo(VERBOSE=TRUE)	# show garbage collection information
object.size(x)	# return number of bytes taken by object x (class is 'object_size')
```

#### I/O
- `read.csv`/`write.csv`: read/write csv files. Slow but with high compatibility

- `save`/`load`: save and load in R binaries. 

  ```R
  save(z,w,file='./zw.Rdata')		# save variables z and w
  save.image(file='./zw.Rdata')	# save all variables 
  load(file='./zw.Rdata')			# load variables in the file. The variables are directly loaded into the names when they are saved. 
  ```

  - **Note**: loading may overwrite an existing variable if they have the same name

- `saveRDS`/`readRDS`: read or save a single object into a file. The object name is not used. 

  ```R
  saveRDS(z,file='./z.rds')
  y <- loadRDS(file='./z.rds')		# won't overwrite
  ```



#### apply

apply functions are broadly used in R. 

- `lapply`: iterates and apply a function. returns a list 
  - **Note**: if the input is a vector, it's considered as a list with the same length (`as.list` for a vector gives a list of length 1). For other types, they're transformed into a list using `as.list` by default. 
- `sapply`: iterates over elements and tries to simplify
- `tapply`: grouped reduce operation (e.g. grouped sum)



#### Misc

- use `str` to convert objects into strings, useful for getting object info 

- `get('a')` get an object with name 'a'

- apply: run a function on all elements of a list 

  ```R
  lapply(l, func)		# returns a list
  sapply(l, func)		# runs lapply and tries to simplify (e.g. list -> vector)
  apply(x,dim,func)	# apply a function to a dimension of a matrix/vector (when x is a matrix, row for dim=1, col for dim=2)
  ```

- `table(a,b,c)` return a spread sheet with counts at each point of the n-dimensional space formed by points in `c(a,b,c)`. The number of vectors are not limited. 

- `duplicated(a)` return a logical vector, with FALSE for any values showing up the first time in `a`, and TRUE for all later appearances. 

- `diff(a)` take difference of elements in `a`. The returned vector is shorter than `a`.

- `jitter` adds noise to data. It's useful when plotting scatter plots with quantization, in which case point density is not visually shown

- `pmatch` can be useful when doing string partial matching. 





## Syntax



#### Conditions and Loops

```R
# conditions
if (a > 1) {
  	print(a) # Note: brackets are required
} else if (a < 0) { 
 	print (0)
}

# a ? b : c
ifelse(a > 0, b, c)

# for loop 
for (i in c(1,2,5)) { xxx }
while (a < 0) { xxx }
```

**Note**: when using `ifelse` function, the output has the same length as the condition vector. If the two value vectors are shorter, the index will be taken by `mod` their length

**Note**: function is a scope, but conditions/loops are not (like python, unlike C). All new variables, including the iterated variable, are visible outside the brackets. 

#### Packages

```R
install.packages('PACKAGE')	# install a package
library('PACKAGE')	# load a package. throws an error if failed
require('PACKAGE')  # load a package. returns FALSE and throws a warning if failed
detack(package:PACKAGE)		# unload a package
```

#### Functions

**Note**: the value of the last expression in the function is returned, if it's not a return statement

```r 
x <- function(var1,var2) { xxxx }	# function definition
y <- function (...) { x(...) }		# function rename using wrappers. y <- x just copy source code of x to y, and doesn't change y if code of x changes
```

**Note**: for named arguments, R uses partial matching by default. 

**Note**: when calling a function, using named arguments before positional arguments is allowed (but bad practice).

## More on functions

##### Components

- `body()`, the code
- `formals()`, the list of arguments
- `environment()`, the location of function's variables (not shown in print if it's the global environment)
- **Note**: primitive functions (`base` package) directly call underlying functions and has none of these components

##### Function scoping

- name lookup first work in the function, then recursively goes one level up to where it's **defined** if not found 
  - Although the lookup happens in the space where the function is defined, it only happens at the time of calling (i.e. dynamic lookup)
- Every function is a environment (i.e. scope)
- To mutate a static variable (variable from upper levels of the function environment), use `<<-`, which looks up for the variable recursively and assign to the existing one found. 
  - **Note** if no variable of the given name is found, a new variable will be defined in the global environment
- Tip: Use `findGlobals()` to find any reference to global variables in a function. 

##### Function call

Every operation is a function call, including `(` and `{`. 

- Use \` to get function definition 

##### Arguments

- Arguments are matched first by exact name (perfect matching), then by prefix matching, and finally by position. 
  - The matching works regardless of whether the argument has a default value. All arguments are matched this way. 
- `do.call(func, args_list)` calls a function using a list of arguments (like python's `func(*args)`)
- default arguments are evaluated **at time of function call**. They can be defined using variables defined in the function (R use lazy evaluation, so the argument is evaluated when they're used for the first time during function call)

##### Multiple arguments

`...` matches all unmatched arguments. Use `list(...)` to make it a list. 

##### Special function

- Infix function: functions with two arguments may be defined and used using `%`: 

  ```R
  `% %` <- function(x,y) paste(x,y)
  'a' % % 'b' # gives 'a b'
  ```

- Replacement function: mutate an object

  ```R
  `second<-` <- function(x,v) {x[2]<-v; x}
  x <- 1:3
  second(x) <- 5L # makes x c(1,5,3), equivalent to "x <- `second<-`(x)"
  ```

  - **Note**: this suggests that "mutation" functions actually makes and returns a copy
    - **Note**: built-in functions implemented using `.Primitive()` will modify in place

##### on exit

- Use `on.exit()` to run codes on exiting the function (return / throw error)
- Use `add=TRUE` for multiple `on.exit()` call, otherwise later ones will overwrite earlier ones

## Stats models

#### formula

Formulae is an object in R that allows the capturing of variables (in the environment where it's *defined*) without evaluating it. 

```r
# creation
y ~ x 			# y depends on x
y ~ x1 + x2		# use + for adding independent variables
y ~ x1 - x2		# use - for removing variables. e.g. -1 for removing offset
y ~ x1:x2		# use : for factors (e.g. for linear regression, this means x1 times x2)
y ~ x1 * x2		# equiv. to x1 + x2 + x1:x2
y ~ I(x1+x2)	# use I() to enclose any terms that should be interpretted as numeric operations
y ~ .			# use . for all other variables in the matrix that's not yet used

# inspecting a formula f
terms(f)		# get everything
all.vars(f)		# get all variable names
update(y~x1, ~.+x2) 	# update formula (update to y ~ x1 + x2)
```



#### models

```r
fit = lm(y ~ X) # linear regression model
fit = lm(attr1 ~ attr2, data=Data) 	# fit Data$attr1 ~ Data$attr2
model = model.frame(formula, data=data) 	# retrieve relevant data into a dataframe without doing any fitting

attributes(fit) # get all attribute listed
fit$coefficients 	# accessing attributes, similar to dot notation in other languages
```



## Plots

```R
plot()			# plot in the current active window (erase whatever was in there)
abline()		# add a line to the current plot. won't erase any existing figure 
dev.new()		# opens a new plot window. = figure() in MATLAB
graphics.off()	# close all plot windows. = close all in MATLAB
par(mfrow=c(3,4))		# subplots filled in row order. Simply use plot command to add plots to the next blank slot
par(mfcol=c(3,4))		# subplots filled in column order. 
layout(mat)				# subplots with indices specified by a integer matrix. Greater flexibity (e.g. merge some cells by giving the same indices)
par(new=T)				# overlap plot in the current active window (don't erase what's original in there)
```

**Note**: 

1. `plot()` erase current plot in the active window. To avoid this behavior, use `point` or `line` for plots. 
2. `par(new=T)` only have plots overlapping without any resize. If the two plots have different labels/axis size/titles, the overlapping strings will get the plot messed up. 



**Tips**

1. When there's lots of points, use pixel for plot (for speed)
2. When the pattern is not clear, do a block mean to reduce point counts
3. Change graphics engine (e.g. disable anti-aliasing) to make plotting faster





## TODO

- memory management of matrix (row/col slice, transpose, copy on modification)
- graphical functions (see end of [this page](https://www.datacamp.com/community/tutorials/r-formula-tutorial))
- Interpretation of `gc()` output
- R performance tips?
- How to write R package and/or import code from another file

