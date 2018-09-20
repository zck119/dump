Basic Notes on Python
=================

**These basic notes were taken a long time ago and doesn't even make sense to me now...** 

## modules

1. use `import` to import certain module (into a module object), then use dot notation to call variables/functions in it

         (e.g. `import math; print math.log10(10)  # (output) 1`)

2. use `from` & `import` to import certain function/variable from amodule 

         (e.g. `from math import pi; print pi # (output) 3.141592653589793`)

 

## Syntax 

### Function

- use `def  ` as key word, a colon at the end of header, and uses indentation to indicate body of the function. Note that argument type is not specified (not restrained to one type). This is like template function in C++.

         (e.g. `def funName(varName):`)

- for variable number of input, use `def funName(var): ` to get all input into a tuple `var`
- to raise an exception, use `raise`



**note**: to restrain the type of input of the function, we can use` isinstance(var,type)` to check that. 

**note**: if a function didn’t return anything, then it return `None`

**note**: if the function involves non-argument variables in `__main__`, the variable can change after function definition. (as if the variable is a static variable for the function). In the function you can only read it or modify it (if mutable), but can give assignment, so it’s better to declare that the variable is the global one using `global var`

**note**: if a function is defined within another function, then its non-argument variables involved is still changeable in `__main__` if they’re also non-argument variables in the other function. If the non-argument variables in this function are arguments in the other function, then their value will replace their position in the definition of the new function.  

 

 ### Condition

use indentation to denote body for each conditional 

```python
# if:
if condition:
         xxx
elif condition:
         xxx
else:
         xxx   

# for:
for loop_var in list_name:
    xxx

# while: 
while condition: 
    xxx
```



## class

definition is simple, without the need to state its attribute: 

                 e.g. `class className(object): // """description """ // methods definition`

Class may have an attribute, and if so, all instances without such attribute automatically have the same attribute (they can have their own, though)



inheritance: `class className(parentClass)`

 

use `hasattr(instance,attr)` to check if a certain instance has a certain attribute.  

use `__init__(self[,…])` as constructor

use `__specialCommand__(self,…)` to overload system operator. 

use `__cmp__(self,other)` to overload all comparison operators(<,>,==,>=,<=)



**note**: for class, `==` is the same as `is`, by default. 

**note**: for methods, the first var in function is the instance that invokes the methods, and methods can be used as `className.function(instanceName,var,…)` or `instanceName.function(var,…)`

**note**: `print` automatically call `str(object)` (built-in), so usually `__str__` is defined. 

**note**: unlike C++, even when we are defining class methods, we need to use `self.attr` instead of just use attr (= `this->attr` in C++)

 

## Types

### Primitive types

`type(var)` returns the type of `var`. 

`int(var)` can convert float & string toint

`str(var)` can convert float & int to string

`raw_input(string)` display the string and takes a line of input, then return the input. 

`copy(var)` returns a **shallow** copy of given var (for instances within the other instance, it only give reference, not shallow copy. i.e., it’s not recursively called to produce an authentic shallow copy. ). It’s in the copy module.

**Note**: all primitive types are immutable in python

 **Note**: function `id(var)` gives the id of a variable, which is used in `is`. For primitive types, variables of same values have the **same** id, so an `is` test is the same as using `==`



### list & tuple

definition:      

- list is defined using `[var,var,var,…]` (not necessarily 
- tuple is defined using `(var,var,var,…)` (not necessarily same 
- dictionary is defined using `{key:item,key:item,…}`. can use `dict[key]=value` to add an item
- set is defined using `{item1, item2, ...}`



index: generate partial list using `list_name[n:m]` (take index n to **m-1**). can omit n for start of list, andomit m for end of list. 

usually use `for i in range(len(list_name))`to traverse the list

 

deleting an element: 

- use `list.pop(index)` to remove element and return it
- `list[index]` to remove element without returning 
- use `list.remove(value)` to remove the first element of value `value`  (gives error if not exist)



aliasing (reference): available for objects(not value), invoked with statements like b = a **(expires upon reassignment of a)** – how to just give b the value of a? can use `b = a[:]` to only give b value of a

 

use * to unpack list or tuple into an array of elements. 

`zip(list1,list2,…)` generates a list of tuples of elements of input lists. **Note** that when the lists are of different size, the results have the same length as the shortest list, not *No* errors are thrown.

`enumerate(list)`  = `zip(range(len(list)),list)`

 

list-string 

- `list(str)` return a list of individual characters of `
- `str.split(delimiter)` returns a list of strings separated from `str` with `delimiter` as delimiter
- `delimiter.join(list)` connects a list of strings with delimiter



tuple-dictionary 

- `dict(list)` turns a list of 2-tuples into adictionary
- `dictName.items()`



sort by certain feature: list of element->list of tuple (feature, element)->sort->get the elements

create a list using for: `[expr(var) for var in list]` or `[expr(var) for var in list if condition]`

 

**Importance of list methods**: list methods return `None` while modify the list itself, and prevent the expiration of reference (`list = list + list2` counts as an assignment and therefore the reference to `list` expires). **Note** that `list1 += list2` is implemented as `list1.extend(list2)` and is thus a mutation.

 

**note**: can use negative indices to indicate counting backwards, like `a = 'abcd'`, then`a[-1]=a[len(a)-1]='d'`

**note**: list and dictionary are mutable (but its length is fixed), while string and tuple are immutable

**note**: list methods all change the list itself and returns nothing. 

**note**: can assign a list or tuple to multiple elements as a,b = [1,2] or a,b = (1,2)

​

# Python Advanced Notes

## Generators and Iterator

#### Iterator

In python loops, a iterator is needed (for iterable objects, built-in `iter` is called to call `__iter__` of the object, which returns an iterator). An iterator is any object with an `next` method and throws `StopIteration` exception. The loop will continue to run `.next()` method until a `StopIteration` exception is thrown. 

`list(iterator)` takes element sequence the iterator returns and makes a list

#### Generator

A generator function is a function with `yield` statement. A generator is always an iterator (it should be considered as an object, with constructor arguments same as function arguments). Therefore, the generator has `.next()` method to run the generator function body until `yield` statement. The generator always resumes running from the last `yield` statement. 

##### Coroutine

The generator can also be used as coroutine: 

```python
def gen_coroutine(num): 
    x = num
    while True: 
        y = yield x  # yield x and wait for input
        x = y or x	 # y = None if there's no input (i.e. running next())
        x += 1
cor = gen_coroutine(10)
print(next(cor))  # 10
print(next(cor))  # 11
cor.send(5)  # y = 5; x = 5; x += 1 (i.e. x = 6). Note that it's executed until the next yield statement
print(next(cor)) # 7 (note that 6 is yield as the return value of send)
```

## Decorator

 A decorator takes a function as input and returns a wrapped function

```python
def decorator(func): 
    def wrapper(*args, **kwargs):
        res = func(*args, **kwargs)
        # do something to res
        return res
    return wrapper

# for convenience, use @decorator before function definition to make it the decorated version of function
```

Use `functools.wraps(func)` to decorate the `wrapper` function so that the decorated function inherits signature, name and docstring from the original function. 

## Context Manager

For any class with function `__init__`, `__enter__` and `__exit__`, it can be used as a context manager: 

```python
with ClassName(args) as xxx: 
    # do stuff
```

`__init__` and `__enter__` are executed before the block, while `__exit__` is executed after the block. It's like a `finally` statement that ensures the execution of `__exit__` in case of exception from the block. 

To handle exception, use `__exit__(self, type, value, traceback)` to handle. The exception is considered handled if it returns True, otherwise it's thrown again and pass to the next level up. 

**Note**: the value `xxx` in `with ClassName(args) as xxx`  is the return value of the `__enter__` method, not the class object. 

A convenient way to define context manager is: 

```python
import contextlib
@contextlib.contextmanager
def func(args):
    # whatever goes into __init__ and __enter__
    yield x	# begins the statements in the 'with' block here. x is assigned to the variable after 'as'
    # whatever goes into __exit__
```



## Class

##### `__init__`

in python inheritance, child class does not automatically execute parent class's `__init__` method, and this has to be manually run. For complex multiple inheritance, it's recommended that all class use `super(CLASSNAME, self).__init__()` to let python determine how to run parent classes' `__init__`. This ensures that all ancestors' `__init__` method are run exactly once. 

##### `@property`

It's a convention to write variable starting with an underscore as private. To execute additional statement when getting/setting a property, `@property` decorator is helpful. e.g. 

```python
class MyClass:
    def set_temperature(self, value): 
        if value < 273.15:
            raise ValueError
        else: 
            self._temperature = value
    def get_temperature(self):
        return self._temperature
  	temperature = property(get_temperature, set_temperature)
   	# this allows using myobj.temperature without setting it below 273.15 by accident

# another equivalent way:
class MyClass: 
    @property
    temperature(self): 
        return self._temperature
    @temperature.setter
    def temperature(self, value):
        # same as before
```

##### `__mro__`

`__mro__` is an attribute of classes that records the parent classes and ancestor classes of the class. When looking for a name (class variable/method resolution) inside a class, the name lookup goes through the list of classes in `__mro__` until there's a match. 

##### Attribute lookup

For attribute lookup of an object, class attribute will also be checked, if the corresponding attribute is not a field of object. In methods, calling `self.attr` may also resolved to the variable of class.

**Note**: value assignment for such a variable using `self.attr = x` won't write on the class attribute. It will only create a copy of the attribute for the object. 

```python
class Test(object):
    x = 1
    def test(self): 
        self.x = 2
        x = 3
a = Test()
a.x		# resolved to Test.x, 1
a.test()
a.x		# resolved to a.x, 2
Test.x	# still 1
```

**Note**: implicit call to magic methods lookup is always done on a higher level: 

```python
class Test(object): 
    def __call__(self): 
        print(1)
a = Test()
a.__call__ = lambda self: print(2)
a()		# resolves to Test.__call__(a), print 1
a.__call__()	# resolve to a.__call__, throws an error as it's not a class method and no argument is given.
```

##### `__getattr__`, `__setattr__` and attribute lookup priority

Like `@property` that allows additional operations when getting/setting a specific attribute, there's also support for additional operations whenever there's an attribute access. 

Priority of attribute lookup (within one object): 

1. First check if there's a `__getattribute__` function defined. If not, it'll call `object.__getattribute__` (`object` is the default parent class of all classes)
   - **Note**: overwriting `__getattribute__()` may cause infinite recursion, since it has the highest priority and is called whenever there's an attribute lookup (including magic method lookup)
   - All following priority are implemented by `object.__getattribute__()`
2. If the object has a data descriptor (a `property` with both getter and setter), return it
3. If the object has an attribute in its `__dict__`, return it. This is the normal attributes we see. 
4. If the object has a non-data descriptor (a `property` with only getter, but no setter), return it
5. Call the object's `__getattr__()` function.

**Note**: I currently have no understanding of the behavior when using an attribute with lower priority in this list to overload an attribute with a higher priority in the parent class (e.g. having a normal attribute in a derived sharing the same name with a data descriptor in its parent class). Do NOT try it.  

## Library

#### `glob`

`glob.glob` searches for files with given pattern and returns a list. Use `glob.iglob` for the corresponding iterator. 

**Note** that it omits directories without read permission without warning. 

**Note** that it omits files starting with dot. Use `os.walk` if dot files are meant to be included. 

#### `collections`

`collections.defaultdict()` gives a dictionary that define a default value for any missing key instead of throwing an exception. 

`collections.Counter()` gives a dictionary for counting. Use `c.update(item)` to update the count of an item. 

`collections.deque()` gives a double-ended queue.

#### `timeit`

- command line interface: `python -m timeit -n num_runs 'python-statement'`

- python interface: 

  ```python
  import timeit
  timer = timeit.Timer(stmt='statement-to-be-timed', setup='setup-statement')
  timer.timeit(count)	# run setup once, stmt count times
  timer.repeat(rep, count)	# run timeit(count) rep times
  ```

#### `numpy`

**broadcasting**: numpy automatically do broadcasting when doing operations on two arrays. It compares shape of matrices and does it automatically if the corresponding dimension of the matrices are the same or 1. Missing dimensions are considered as dimension 1. The dimension matching goes *from right to left*. 

`numpy.vectorize(func)`: a decorator that turns a function on scalar to one on numpy array (element-wise operation)

`numpy.searchsorted(data, query)`: find the indices for each element of `query` such that `data` remains sorted if these elements are inserted at these indices. `data` **must** be sorted before calling the function. 

`numpy.where(condition, x, y)`: return a matrix with values from `x` if condition is true, otherwise `y`

**note**: `numpy.concatenate()` makes a copy



#### `logging`

`logging` is a builtin python library used for logging events in python process. 

How `logging` works:

-  each log event has a level of importance. 
  - predefined ones are `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`. 

- loggers: a hierarchy of loggers are used. 
  - Each event propagate iteratively to its parent logger (set `logger.propagate=False` to disable it). 
  - The hierarchy is determined by the name of loggers (dot notation for child of a parent logger)
  - A logger has its own level. events below the logger's level won't be sent to its handlers. 
  - It's recommended that each library sets its own logger (using `logging.getLogger('name')`), so we can track where each logging event is generated. 
- Each logger can have multiple handlers. Each handler can generate a record for each event passed to that logger. 
  - Each handler can have its own level, formatter and destination (e.g. stdout, file)
  - A handler has its own level. events below the handler's level won't be handled (i.e. generate corresponding record) even if it's sent from its logger. 

#### `h5py`

Hdf5 is a useful data format that supports fast data IO and slicing. 

Each hdf5 file is like a filesystem, consisting of groups (similar to folders) that consist of other groups and datasets (similar to a file, which stores arrays of data of same datatype). The name of each dataset/group has slashes indicating hierarchical relationships, like paths in filesystems. 

Each group/dataset can have its own attributes (metadata). This is pretty useful to store values that describe the dataset. 

```python
import h5py 
import pandas as pd
import numpy as np

# Using h5py directly
f = h5py.File('test.h5', 'w')
f['x'] = np.random.rand(5,10)  # save a np array as a dataset with name '/x'
f['a/b/c'] = np.random.rand(2,3)  # save as a dataset, while also creating intermediate groups
f['a'].attrs['test'] = True  # set the attribute on group 'a'

# using pandas 
df = pd.DataFrame(np.random.rand(8,10))
store = pd.HDFStore('test2.h5')
store.put('testdata', df)
store.get_storer('testdata').attrs.test = True  # set attribute on group

# using pandas if no metadata is required
df.to_hdf('test3.h5', 'testdata')
```

**Note**: original `h5py` library follows the hdf5 format closely. `pandas.HDFStore` supports more flexibility through `pytables` (built on top of `h5py`), enabling features like **using dictionary in metadata.**  



## Library - `Pandas`

[link](tomaugspurger.github.io) seems like a good place for tips. 

### Basics

```python
import numpy as np
import pandas as pd

# declare
s = pd.Series(some_numpy_array, name='name')
df = pd.DataFrame(some_numpy_matrix, index='rownames', columns='colnames')

# get names 
df.index	# row names
df.index.get_level_values(level_name)	# for multi-level index, get list of row names for a specified level

# indexing/slicing
# by name (NOT index), only on one dim: 
df[index1:index2]			# row slicing
df[col1]				    # col slicing with one col (NOT available for row)
df[[col1, col2]]			# col slicing
# by name (NOT index), on two dim:
df.loc[[index1, index2], [col1, col2]]
# access multi-index 
df.loc[(index1, subindex1)]		
# by row/col number
df.iloc[0:4, 2:5]
# by col name
df.<col_name> # note that if column name coincides with df's builtin function, the function will be returned, instead of the column

# multiple level of index
df.loc[pd.IndexSlice[:,['l1','l2']], :] # for selecting all rows with level 2 label being l1 or l2

# index change
df.index = pd.to_datetime(df.index)		# use datetime type for index
df.set_index(colname)	# set a column as index instead of default number
df.reset_index(drop, inplace)	# reset index as default numbers. 
							# drop: delete current index instead of keeping it as a column
    						# inplace: change index in place instead of creating a new one
        
df.reindex(rownames)	# conform index as given. Note that this is not a reset index operation and each row will preserve their indices. This operation changes order and may add empty rows. 
					# note that if rownames is a MultiIndex, df can choose to align with a specific level, and data may be broadcast to fit in the index. 
   
# rename columns
df.rename(columns=lambda x: xxx)
df.add_suffix(xxx)
df.add_prefix(xxx)

```

**Note**: string indexing is available for rows/columns. 

**Note**: `loc` slicing keeps **both** end points. `iloc` only keep the left end, similar to most python syntax. 

**Note**: index keys/column keys may have the same name, in which case multi-index or number keys are recommended. 

**Note**: when using `loc`, list is used for getting indices of the same level, while tuples are used for multi-level indexing (i.e. `x.loc[[0,1]]` means rows with index 0 and 1 at the top index level, while `x.loc[(0,1)]` means rows with index 0 at the top level and then index 1 at the next level). `iloc` has no such problem, since it uses row number and is not affected by multi-indexing

### Indexing Note - View or Copy

Pandas does not guarantee return value of indexing to be a view or a copy ([link](https://stackoverflow.com/questions/23296282/what-rules-does-pandas-use-to-generate-a-view-vs-a-copy)). Rules (subsequent override)

- All operations generate a copy
- If `inplace=True` is provided, it will modify in-place; only some operations support this
- An indexer that sets, e.g. `.loc/.ix/.iloc/.iat/.at` will set inplace.
- An indexer that gets on a single-dtyped object is almost always a view (depending on the memory layout it may not be that's why this is not reliable). This is mainly for efficiency. (the example from above is for `.query`; this will **always** return a copy as its evaluated by `numexpr`)
- An indexer that gets on a multiple-dtyped object is always a copy.

As such, it's **unsafe** to use chained assignment (assignment without chain is guaranteed). Use one indexing operation if possible. Warnings are issued when possible bugs exist. 

- If a data is accessed multiple times, make a copy using `.copy()`(this also makes sure of good memory management). 
- If only a slice is required, discard the original dataframe (like an ownership transfer). This won't trigger the warning

Whenever the indices are messed up, it's *probably* a copy. 

### Data transformation

```python
# spread sheet
spreadsheet = df.pivot_table(values='value_col', index='index_col', columns='column_col')
	# create a spreadsheet using the values, index and columns (convert a long list of entries into spreadsheet info). index/columns can be list of names in order to get multi-level index for row/index. 
    # aggregate function can be specified to group values with the same index/column, e.g. sum, len, ... default is taking the mean
    
# stack/unstack
df.stack()			# rotates lowest level of column MultiIndex to lowest index
df.unstack()		# rotates lowest level of index MultiIndex to lowest column 
grouped = df.groupby(level=levelname, axis=n)		# return a grouped object for aggregation
grouped.get_group(group_name)		# get a group by name
grouped.groups	# get a dictionary from group key to full index/row tuple  

# cross section 
df.xs(('name1','name2',...), level=('level1','level2',...),axis=1)
	# get the cross section for certain name at the specified level. Useful in multi-index dataframes, when the same name is repeated in lower levels
```

### Merging

For `pd.merge`: 

- `how`: 
  - left/right: keep rows in the first/second dataframe; 
  - outer: keep rows in either dataframes (add `NaN`)
  - inner: keep rows in both dataframes
- `on`: columns used to merge as join key. Use `left_on` and `right_on` if they have different names. Use `left_index` and `right_index` if the key is the index instead of a column

`pd.concat` is also available for merging (more numpy style). 

Fillin NaN: 

```python
s.map(element_map)			# map element according to a dictionary. the result is NaN is the element is not in the map
s.fillna(na_filled_series)	 # set the NaN in the series as the second series
s.fillna(method='pad', limit=n)	# method=pad/bfill: copy the last/next non-NaN value
							# limit: for each gap of NaN, only fill the first/last n (depending on method)
s.replace(to_replace=newvalue, value=oldvalue)	# replace element
s.dropna()					# remove rows with NaN
s.interpolate()				# use interpolation to fillin na. Default fills in mean. Use method='values' or method='time' to make use of nearset non-missing row indices
```

**Note**: Pandas index is immutable, so adding rows/merging rows means creating a new object (slow). 

### Data checking functions:

```python
df.describe()		# return description of the dataframe (count, mean, std, ...)
df.info()			# basic information 
df.head(n)			# return first n rows of data
df.tail(n)			# return last n rows of data
df.set_option('precision', n)	# set data print precision
s.isnull()			# return a boolean Series with True where the element is NaN 
```

**Note**:

-  `np.nan` is used for `NaN` values, and `NaT` is used for missing datetime types. It's a float type, so columns with missing values can't be integer types - they are converted to float if numeric, or `object` type if mixed with strings or **boolean**
-  `None==None` returns True, but `np.nan==np.nan` returns False. 
-  `NaN` are ignored with the column/row is used by `groupby`. 
-  `df.dropna()` drops rows with `NaN`. Use `thres` argument to drop only rows with more `NaN` than `thres`. 

### Grouping

pandas supports grouping data according to index/column and perform operation on each group. 

```python
gp = df.groupby("age")  # create a group object based on age
gp = df.groupby(pd.grouper(xxx))  # for custom group, e.g. ranged group, time interval group
gp = df.groupby(["age", "sex"])  # group by multiple columns

# per group operation 
df_normalized = gp.apply(lambda x: x - x.mean())  # apply a function to each group
df_doubled = gp.transform(lambda x: 2*x)  # apply a transformation to each group
df_mean = gp.agg("mean") # aggregate each group to a value. Note that np.mean or lambda function also works. 
						# gp.agg() may take a list of functions to generate multiple columns per group
```

Any columns used in `groupby()` will be added as a level of index in results. 

**Note**: in `apply()` the function works on each group as if it's a slice of dataframe. In `transform()` the function works on **each column** of each group separately, so it doesn't have the flexibility like `apply()` to do operations on multiple columns / add new columns. 

**Note**: `apply()` may be used applied twice on the first group, so that pandas can choose which implementation of `apply()` it'll use. Don't use functions that run differently at each time. 



### Time series

```python
df.diff(n) # discrete difference. the first n elements are set to NaN
df.shift(n) # create a shifted version of df

# Useful grouping methods. All can be used to apply customized functions (.apply(func))
df.rolling(n) # rolling window of size n
df.resample(t) # strings like "1h" are also supported as time interval. Both downsampling and upsampling are supported. 
df.expanding() # each group is all the elements up to this point. 
df.ewm()    # exponentially weighted moving window. Limited supported functions. May NOT work with customized functions. 
```

All window size can be integer or time interval (called [offset](https://pandas.pydata.org/pandas-docs/stable/timeseries.html#dateoffset-objects) in pandas). strings may be automatically converted to offset. 

### Performance tips

- Don't use different types in the same column - disables any type-specific optimization

- Use `nlargest/nsmallest` to run linear-time element selection instead of sorting

- in underlying storage, pandas dataframe merges columns of the same type into matrix and save each matrix individually. Empirically, getting a column is faster than getting a row

- When merging multiple dataframes, it's always faster to run `concat` once instead of `concat` them to a large dataframe one by one. 

- Unwrapping an operation into vectorized functions are faster

  ```python
  # slow
  df.groupby('age').v1.apply(lambda x: x - x.mean())
  # faster
  df.v1 - df.groupby('age').v1.transform('mean')
  ```

  - **Note**: when using vectorized functions, and if pandas can identify that a vectorized function is being applied, `transform()` is much faster than `apply()`. Otherwise their performance is comparable. 

- the `csv` library (builtin) is more efficient than `pandas.read_csv()`. Use it if you don't need a dataframe returned. 

### Misc Notes

- For any operations on dataframes/series, the values are aligned based on index.



## Performance Boost

Contributing factors for Python's poor performance: 

1. Dynamic typing, which involves substantial overhead compared to typed languages like C/Java
2. `is` is faster than `==`, since it only uses memory location instead of running a `__eq__` function

### Cython

Cython is a package used to compile python codes with user-determined types, thus using type-specific optimization to boost performance. The source code can be simply python code (no optimization, and no speedup), and can have specific type requirements for specific speed up. 

Cython gives a pretty good performance boost, but writing Cython code is pretty time-consuming. 

```cython
cdef int n  # declare variable
cdef int p[1000]  # declare static array 

# cython supports cpp STL. We'll probably use numpy. 
from libcpp.vector cimport vector
cdef vector[int] p
p.reserve(100)  # declare and allocate memory for cpp STL vector
p.size()  # size of vector
p.push_back(2)  # append to vector

```

Declaration can use: 

- `def`: pure python. safe to use. assumes every line is python
- `cdef`: pure C. all variables must be typed. Functions defined with `cdef` are not visible in Python. 
- `cpdef`: middle ground of `def` and `cdef`. 

#### Class in Cython

```cython
cdef class TestClass:
    cdef double x  # object variable
    def __cinit__(self): 
        self.x = 1 
    def __dealloc__(self):  # destructor
        pass
```

- `__cinit__` is called on object construction, before the object is constructed, so at this point `self` is not fully constructed yet and we do nothing with it except assigning to its `cdef` fields. 
- If there's any parameter in `__cinit__`, then any `__init__` of classes (including its derived classes) must match the argument list. 

- A `.pxd` file with class and function signature can be used so that other cython files can import these function signatures. 
- Use `inline` after `cdef` to make function inline
- Use type `Py_ssize_t` for python array indices and numpy array indices/shapes.
- Use `# cython: infer_types=True` at the beginning to have cython infer data types (like iterating indices in `for i in range(10)`)

#### Caveats

- Careful about unsigned types. `cdef unsigned n=10; range(-n,n)` is empty since `-n` is a large positive integer (unsigned underflow)
- python `float` wraps C `double`, `int` in Python2 wraps C `long`. 

#### Profiling

[Toturial](http://docs.cython.org/en/latest/src/tutorial/profiling_tutorial.html)

- Use `# cython: profile=True` at the top to enable profiling (otherwise `cProfile` won't give meaningful results)
- Additional directive available for more detailed profiling
- Note that inline functions still incur function overhead during profiling. Use `@cython.profile(False)` to decorate inline functions to disable profiling on inline functions. 

Sample: 

```python
# profile.py

import pstats, cProfile

import pyximport
pyximport.install()

import calc_pi  # cython module

cProfile.runctx("calc_pi.approx_pi()", globals(), locals(), "Profile.prof")  # calc_pi.approx_pi() is the function being profiled

s = pstats.Stats("Profile.prof")
s.strip_dirs().sort_stats("time").print_stats()
```

#### String

Due to string types in Python 2 and Python 3, the use of strings in Cython is a little tricky. Avoid if possible. ([Tutorial](http://docs.cython.org/en/latest/src/tutorial/strings.html))

#### Numpy

Cython supports two ways of numpy access: explicit numpy array declaration or typed memoryview (*recommended, faster*). 

##### Typed Memoryview

```cython
cimport cython
@cython.boundscheck(False)  # Used to: Deactivate bounds checking
@cython.wraparound(False)   # Used to: Deactivate negative indexing.
def compute(int[:, :] arr):
    result = np.zeros((4,5), dtype=np.intc)
    cdef int[:, :] result_view = result # use memoryview for faster indexing
    # do stuff
    return result
```

- Typed memoryview provides a pointer to numpy arrays, and fast indexing (since it's using C indexing)

- Note that all indices / shapes should have type of `Py_ssize_t` (`int` can also work, though), including iterating indices in `range()`. 

- Declare an array as contiguous gives extra speedup. Use `[:,:,::1]` for C-contiguous 3D array, and `[::1,:,:]` for Fortran-contiguous 3D array. 

- Use fused types to write function for multiple types (like cpp templates)

  ```cython
  ctypedef fused my_type:
      int
      double
  def compute(my_type[:, ::1]): 
      # do stuff
  ```

- Slicing memory view is available (`cdef int[1:50:2,:] x = arr`)
- **Note**: `None` is allowed as input for memory view. Use `int [:,:] view not None` to indicate that an input array is not None.  
- Same as numpy, use `None` as index to add new axis
- `const` can be used for read-only memoryview

##### Typed Numpy array

- requires GIL while `cdef np.ndarray` (memoryview doesn't need it)
- Need `cimport numpy as np` at the top for compile-time information
- For every type in numpy, there's a type with `_t` suffix. The type without `_t` are python types and can be used as numpy `dtype`, while the type with `_t` are the corresponding underlying type (C type identifier) that can be used in `cdef` or defining types in `np.ndarray[dtype_t, ndim]` (requires `cimport numpy`. These two cannot interchange in cython. 
  - See [link](https://stackoverflow.com/questions/21851985/difference-between-np-int-np-int-int-and-np-int-t-in-cython) for details.

```cython
import numpy as np
cimport numpy as np
DTYPE = np.int  # used in python types, e.g. numpy functions' dtype argument
ctypedef np.int_t DTYPE_t  # used in cython types

def naive_convolve(np.ndarray[DTYPE_t, ndim=2] f):
    assert f.dtype == DTYPE
    cdef DTYPE_t value
    # do stuff
```



### Numba

Numba is a package used to automatically vectorize functions using user-specified types for all arguments. Its writing is much easier (just use decorator), and gives a good performance boost (not as high as Cython, of course, but on the same order). It's usually used for short critical codes. 

Pros: 

- easy to write code, more readable than Cython
- decent performance boost compared to Python

Limits: 

- Numba only accelerates code that uses scalars or (N-dimensional) arrays. You can’t use built-in types like `list` or `dict` or your own custom classes.
- You can’t [allocate new arrays](https://github.com/numba/numba/pull/719) in accelerated code.
- You can’t use [recursion](https://github.com/numba/numba/pull/719).

```python
import numba
@numba.jit()		# defer compilation until first use

```

### Profiling

For profiling, basic tools like `timeit`, `line_profiler` and `memory_profiler` are available (all supported in IPython/jupyter notebook). 

`pyrasite` is a cool tool that injects low level commands into a python process to access its variables. All that's required is the pid of the python process. 

`snakeviz` provides good visualization for the profiling results. 

### Testing

`pytest` is generally recommended for unit testing in python. 

`pytest --cov` can be used to test for coverage. 

`testfixtures` library is a useful library that provides convenient wrapper for most `pytest` tests. 



### 

## Misc notes

1. default arguments are only instantiated at the time of definition. Don't use **mutable** type.

   - An application: 

   ```python
   for x in range(3):
       def func(): 
           return x
       funcs.append(func)
   results = [f() for f in funcs] # [2,2,2]
   
   for x in range(3):
       def func(x=x): 
           return x
       funcs.append(func)
   results = [f() for f in funcs] # [0,1,2]
   ```

2. in try-except-finally statement, a block stops when fully executed/reaches a `return` statement, then the finally block is executed (even when try block has `return` statement). If finally block has a `return` statement, all previous `return` statement are **overwritten**; otherwise return value of pervious blocks are returned. 

3. `isinstance(var, type)` check if `var` is of type `type`. For classes, it returns True if `var` is a class inherited from class `type`.  

4. python uses `**` as exponential operation, not `^` (C++ uses `power(x,n)` in `cmath`).

5. python performs floor division forint (`1/3=0, -1/3=-1`), but for type conversion (float->int), value always gets closer to 0. (**python2**)

6. python supports taking mod with neg/float. For `x%y`, the sign of result is the same as that of `y`. 

7. in interactive mode, result is automatically printed, while in script mode, each output needs to be specified by word `print` up front. 

8. `+`and `*` works with strings for as `strcat` and `repetition`

9. python math functions uses rad(like sin)

10. the `None` type is not displayed even in interactive mode, unless using `print`.

11. variables declared in loops are accessible **outside** loops (like Matlab, as compared against C++), posing potential  

12. call for `repr()` in string: 

    - For `%` method: `"%r" % obj`
    - For `str.format()` method: `{x!r}.format(x=obj)`

13. `dict.keys()` is a dynamic object. The dictionary can be updated after `dict.keys()` is created (but not between it's used)

14. Use `r'pattern'` for regex patterns so that backslash are not lost. 

15. For python subprocess to communicate: 

    - `subprocess.PIPE` can be used, but may lead to locking due to full buffer
    - `p.communicate()` puts all stdout in memory (no locking), but may need use lots of memory. 

16. library `rapidjson` provides faster json parsing than `json`. 

17. When using `argparse`, the `type` argument is actually used as a preprocess function on the input string, so customized preprocess functions can be used here. 

18. `tempfile.NamedTemporaryFile` can be used to create an actual temp file accessible by other threads. Note that it's removed after closed. 


## Migration to Python 3

http://sebastianraschka.com/Articles/2014_python_2_3_key_diff.html

http://www.asmeurer.com/python3-presentation/python3-presentation.pdf

**Note**: Python 3 is generally slower than Python 2

1. `print` is a function now. For printing without newline, use `print(a,end='')`

2. **Use `//` for integer division**

3. use `range()` instead of `xrange`

4. Use `raise Error('msg')` to raise error with messages

5. Use `next(gen)` for generator next loop (also works in python 2)

6. **List comprehension variables no longer leak into global variable scope (for loop iterated variable still leaks)**

7. Many builtin functions return iterable objects instead of lists, including `map(), filter(), zip()`, and `.keys(), .items(), .values()` of dictionary

8. Rounding to even numbers in case of ties (.5), i.e. `round(0.5) # = 0`

9. Unpacking can use lists, e.g. `a,b,*rest=range(10)`

10. Keyword only arguments: optional arguments can now appear after variable number of arguments. These optional arguments must be specified by keyword and not by position

  ```python
  def f(a,*args,debug=False):	# debug has to be explicitly specified by debug=True as variable number of arguments are given to *args
    pass
  
  # to prevent using optional arguments by position
  def f(a, b, *, debug=False): 
    # function takes 2 positional arguments. f(1,2) and f(1,2,debug=True) would work, while f(1,2,True) would raise an error
    pass
  ```

  ​

11. Subgenerators using `yield from` keywords. e.g. `yield from range(10)` is equivalent to `for i in range(10); yield i`

12. Annotations are available but **not enforced**. e.g. 

    ```python
    def f(x: int) -> float: 
      return float(x)
    
    f.__annotations__	# returns {'return': float, 'x': int}
    ```

13. `__matmul__` is equivalent to `@` now, i.e. `np.dot(A,B)` is equivalent to `A @ B`

14. `Path` class can be used for path handling 

    ```python
    from pathlib import Path
    directory = Path("/etc")
    filepath = directory / "test_file.txt"
    if filepath.exists():
    	stuff
    ```

15. `dict` in python3 is actually ordered. The difference with `collections.OrderedDict` is that it doesn't care about key order when checking equality. 

## Notes from [wtfpython](https://github.com/satwikkansal/wtfpython)

1. String identity 

   - CPython optimization may reuse existing strings to save memory, leading to different immutable objects with same value having the same id. 
     - The correspondence between immutable object's value and its id is not fixed. 
     - This optimization also exists for numbers between -5 and 256. Python pre-allocates memory for these values and when they're used, the program always gets a reference to them. 
   - Only strings satisfying certain rules are affected by the rule above.

2. As an optimization, **immutable variables** (**not only strings**) assigned to the same value at the same part of program compiled are assigned to the same memory (i.e. reference to the same object). For interactive environment, each line is compiled separately; for `.py` files, each file is compiled as a whole. 

3. The `id` of an object is its memory location, so it's only unique *for the lifetime of the object*. 

4. In a [generator](https://wiki.python.org/moin/Generators) expression, the `in` clause is evaluated at declaration time, but the conditional clause is evaluated at runtime, thus the following: 

   ```python
   arr = [1,2,3]
   g = (x for x in arr if x in arr)
   arr = [0,2,4]
   list(g) # [2]
   # the 'in arr' is replaced by 'in [1,2,3]', but the conditional clause is evaluated after arr is re-assigned. 
   ```

5. `is not` is a single binary operator.

6. In python, raw literal (`r'some_string'`) works by changing the behavior of backslash and having it passing itself and the next character through, so in a raw literal, backslash cannot be the last character (otherwise the quotation mark will be passed ask part of string)

7. Python allows implicit string concatenation: `'hello'"world"` is the same as `'helloworld'`

8. Booleans are a subclass of `int` (so `isinstance(True, int)` gives `True`). 

9. The `+=` operator modifies the **mutable** object in-place without creating a new object. For immutable objects, it's the same as expanding `a += x` into `a = a + x`

10. `dis.dis(compile('<code>', '', 'exec'))` can be used to read compiled code. 

11. Exception variable is cleared at the end of except clause, so don't have another variable using the same name in the same scope. 

    ```python
    except E as N: 
        foo
    # is equivalent to: 
    except E as N:
        try: 
            foo
        finally: 
            del N # any variable with name N in this scope is cleared after the execution of except clause
    ```

12. If two objects hash to the same value (`__hash__` gives same results) and are equal (`__eq__` returns `True`), they are represented by the same key in the dictionary (even though they may be from different class).

13. Assignments work by first evaluating the expression (**only once**), then assigning to each target list (delimited by `=`) **from left to right**

    - So this works: `a, b = a[b] = {}, 5`, since Python first assigns `a,b = {},5`, then `a[b] = {},5`. Note that the expression is only evaluated once, so here the `{}` is actually `a` already. This makes `a` into `{5: ({...}, 5)} `.

14. **chained operations are inherently connected by `and`**, i.e. `a op1 b op2 c` is equivalent to `(a op1 b) and (b op2 c)`. Note that although we have this syntax, each expression is still executed **only once**. 

    ```python
    False is False is False # True, equiv. to (False is False) and (False is False)
    ```

15. Scopes nested inside class definition ignore names bound at the class level.

    ```python
    x = 1
    class C:
        x = 2
        y = (x for i in range(2))
    print(list(C.y)[0]) # gives 1
    ```

16. `else` clause: 

    - The `else` clause after a loop is executed only when there's no explicit `break` after all the iterations.
    - `else` clause after try block is also called "completion clause" as reaching the `else` clause in a `try` statement means that the try block actually completed successfully.

17. In Python, the interpreter modifies (mangles) the class member names starting with `__` (double underscore) and not ending with more than one trailing underscore by adding `_NameOfTheClass` in front.











