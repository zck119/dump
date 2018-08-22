Notes for C/C++ Programing 
===============

## C

**Primitive types**
 - char, short, int, long (number of bytes not guaranteed)
 - `int32_t, int64_t` (`#include <stdint.h>`)
 - float, double 
 - complex

#### Struct
Use `typedef struct {...} STRUCT_NAME ;` to omit the `struct` keyword when referring to the struct name, otherwise declaration has to have `struct` at the beginning. 

e.g. 
```
struct Matrix {
    int n, m;
};
struct Matrix A;
```
or 
``` 
typedef struct {
    int n, m;
} Matrix; 
Matrix A;  
```

#### Pointers & Memory Management
Unlike Java, C copies values as program arguments, which is quite inefficient. 

**Note** `malloc` takes time, so keep it out of loops, and as outer recurses as possible.

**`restrict`** 
The `restrict` keyword means that for the lifetime of this pointer, any content related to this pointer is only refered to using the pointer, not any other pointers. This enables some optimization (e.g. a variable associated with a pointer only needs to be loaded once if known it's not updated using other pointers). 

#### Headers and Source files
xxx.h: 
```C
#ifndef XXX_H
#define XXX_H
// constants
extern int CONST_NAME;

// struct definitions
typedef struct {
    int ...;
    union {     // Unamed union/struct members can be directly accessed
        int a;  
        char b;
    }
} xxx;

// function declarations (not definitions)
int xxx(...);

#endif
```

xxx.c:
```C
// constants
int CONST_NAME = ...;

// struct initialization 
xxx a = {b,c,...}; // or one member a line 

// function definitions
int xxx(...) {
    ...
}
```

**Note** no definition should be included in header files. For constants, use `extern`. 

#### Macros
Macros are substituted into source code directly. Could cause problems. 
Alternative: const variables and static inline functions. 

#### Coding style
 - No space at the end of line 
 - System headers should be included first, then your own headers. 

#### Common functions 
##### IO
- `strtod(str, endptr)`: parse a string into a double, setting endptr pointing to right after ending of the number. `errno` is set to `ERANGE` if parse failed. 
- `strtol(str, endptr, base)`: same as `strtod`, except with a base setting (usually 2, 8,10 or 16) 
- `puts`: print a string with a newline.
- `fscanf(file,pattern,addr)`: given a file handle (returned by `fopen`), scan the next few characters with the given pattern, save in the following addr(s). **Note** that `%s` will read until the next space. 
  - **Note** `fscanf` reads until a whitespace, leaving the whitespace in buffer. However, it removes leading whitespace when reading (**expect** for `%c`).  
- `fread(ptr,size,count,file)`: read `size*count` bytes from file into ptr.  
- `fseek(file,offset,origin)`: set position indicator of file handle to origin+offset. origin is one of `SEEK_CUR`(current), `SEEK_END`(end), `SEEK_SET`(begining). 
- `ftell(file)`: return position indicator offset of file handle.
- `fgetc(file)`: read a character from file handle. returns `EOF` when reaching end of file. 
-`getchar()`: read a character from stdin.  


## C++
### Memory Management

Syntax: 
```C++
int *p = new int(2);    // initialize to value 2
int *p = new MyClass(2); // initialize using MyClass constructor with argument 2

int *p2 = new int[5];    // no initialization
int *p2 = new int[5]();  // initialization all elements to 0
int *p2 = new int[5] {n1,n2,n3,n4,n5};  // initialize all elements, C++11 feature

delete p; 
delete [] p2;
```

`new` and `delete` replace `malloc` and `free` of C. 
- Pro: 
    1. They ensure that constructor and destructor of the class is called. Thus, they should be used for all non-POD types. (POD means plain old data, i.e. class/struct with no constructor, destructor and virtual member function)
    2. They throw `std::bad_alloc` error instead of return `NULL`
- Con: They don't have a counterpart to `realloc`
- Note: the two pairs should not be mixed

### Type case & RTTI

`static_cast`: always preserves address. Useful for casting to/from `void*` or primitive type conversion. *No* type checking when casting. 

`const_cast`: remove/add `const` modifier for a variable. Useful when removing `const` modifier for `const` argument reference / method. Note that using it to generate a non-const reference to an *underlying* const variable results in undefined behavior. 

`dynamic_cast`: casting polymorphic types (classes with virtual methods). Can be used to cast base class pointer to derived class objects to derived class pointer. Returns `nullptr` if casting fails. 

`reinterpret_cast`: if a pointer is casted to a different type, `reinterpret_cast` guarantees casting it back with the same address. Basically allows casting between any two types. Most *dangerous* type cast. 

### Classes

Type conversion is allowed, if the class has a constructor with only one argument with the same type. To avoid implicit type conversion, use `explicit` when declaring such constructors. 

##### Initialization List
Syntax: immediately follows the constructor signature, before the function block, with `var(value)` pairs delimited by comma. Variables are initialized in the order they are defined in the class definition, NOT the order they appear in the list. 

Pro: 
1. In inheritance, it allows choosing the constructor of parent class instead of call the default constructor of the parent class
2. The only way to initialize const and reference, since they're immutable

**Note** that C++11 promotes the use of `{}` instead of `()` when calling the constructor, in order to avoid confusing the compiler about constructor and function declaration. For vector, `{ {...}, {...}, {...}}` is acceptible. 

##### Default member functions 

**C++98/03**: 
1. default constructor
2. copy constructor 
3. copy assignment operator
4. destructor
  **C++11**:
5. Move constructor
6. Move assignment operator

**Note**: in practice, either all or none of the destructor, copy constructor and copy assignment operator should be defined. If any one of them are defined, all others are not automatically defined. Use `=delete` to disable the default method, and `=default` to enable the default method when not automatically defined. 

##### Constants
- Class-wide constant should be declared within class declaration (.h file) with `static const` identifier, and defined in the .cpp file (outside constructor). The only exception is `int` type, which can be defined directly in the class declaration. 
- Object-wide constant should be declared within class declaration and defined using initialization list
- Object-wide constant array is not supported, but there's a workaround using `const vector<T>`
- `const` binds to left unless it can't, then right. 
- use `const` at the end of class method declaration to restrict the method from modifying class variables

##### Keywords

- `protected`: make variables only accessible in itself/derived classes
- `mutable`: make the variable mutable even when the calling method is `const` modifier
- `inline`: make the method inline. The function should be defined in the header file, not the .cpp file. 

##### Template Class

Syntax: 
```C++
// .h
template<typename T>
class MyClass {
    T xxx; 
    T func(); 
}

template<typename T> T MyClass<T>::func() { return xxx; }   // general function definition
template<> char MyClass<char>::func() { return xxx; }   // specialization for T=char case, for this one function only
```

**Note**: Compiler only instantiates a template class to a real class if it's used. Therefore defining class methods in .cpp file won't actually compile class methods. There're two way to solve this: 
1. define class methods in a separate file (e.g. .tpp file) and include it at end of .h file
2. define class methods in a .cpp file as usual, generate instantiation explicitly at end of .cpp file (e.g.`template class MyClass<int>`). However, doing so will only enable use of template class with explicitly *instantiated* types. 

**Specialization**: Give special definition for template class with some specific type. 
```C++
// full specialization
template<>
class MyClass<int> {
    int a;
    void func();
}

void MyClass<int>::func() { } // template<> not required here
```

### Inheritance

A base class pointer/reference can point to a derived class object. In this case, the object is treated as a basic class object and basic class methods are used. 

##### Virtual inheritance

To ensure a base class pointer pointing to derived class object use derived class methods, the method of both needs to be declared `virtual`. 

- a base class virtual method makes the same method in derived class virtual
- `override` can be used in derived class virtual method declaration to ensure it's shadowing a base class method (avoid typo)
- Add `=0` to any method signature to make the base class pure virtual - unable to have objects, compel all derived class to override this method (the base cannot define this method). Common when using base class as an interface. 



###  STL Containers

#### Iterator

Each STL container has its corresponding iterators that operates like a pointer of an array (i.e. use `*iter` or `iter->func()`). 

```C++
vec.begin()			// iterator at the beginning
vec.end()			// iterator at pass-end 
  
vec.rbegin()		// iterator at the end 
vec.rend()			// iterator at pass-beginning
```

**Note** that reverse iterator moves backwards (i.e. `for (riter=vec.rbegin(); riter != vec.rend(); ++riter))` iterates over a vector in reverse order). 

#### sort & binary_search

`std::sort(start_iter, end_iter)` sorts a given array defined by the begin/pass-end iterators. The sort is inplace, but not stable. 

For `sort` to run, operator < must be defined for the type in the vector. An anonymous function can also be used for defining the comparison function. 

- Anonymous function: `[ ](typename var1, typename var2, ...) {...}`. Whatever in `[ ]` is the capture, i.e. variables outside of function that needs to be accessed within. Use `=` for value copy, and `&` for reference. Omit variable names (`[=]` instead of `[=x1,&x2]`) to capture every variable needed automatically. 

`std::lower_bound(start_iter, end_iter, value)` and `str::upper_bound(start_iter, end_iter, value)` uses binary search on a sorted array `[start_iter, end_iter)`. 

- **Note**: They return the first/past-last iterator of values no *less/greater* than value. This means that even if the value is not found, the returned iterator may not be `end()/begin()`. To check for results, use `(it != vec.end() && (*it) == value)` for `lower_bound`. 

#### vector `std::vector<typename, allocator>`

If the allocator is omitted (usually), a default allocator is used. 

`capacity()`: the size of the underlying buffer (not the vector size)

`resize(n)`: resize the vector to size `n`. If the new size is greater than before, all new elements are assigned using hte default allocator. 

**Note**: vector does memory management itself, and it allocates new memory and copy all objects when capacity is reached. The copy operation uses the copy constructor, or the move constructor if defined (C++11, to boost performance for these cases). All element assignment (e.g. `push_back()`) also uses the copy constructor / move constructor. 

#### `pair`

wraps two elements in one object. Used for `map`. 

#### `set`

`auto it = s.find(val)`: find an element in the set. `it != s.end()` checks if the value is found. 

#### `map`

The iterator of `map` is like  a `pair` that has `iter->first` being the key and `iter->second` being the value. 

`m.insert(pair<...,...>(key, value))`: insert `m[key]=value`. 

**Note**: `insert` fails if `key` already exists in the map. 

**Note**: for missing keys, `m[i]` returns a value constructed using default constructor.

**Note**: map keys are sorted in a binary search tree, and iterating using its iterator always go through keys in non-descending order (non-ascending order for `rbegin()`). However, due to the nature of binary search tree, this iteration does not result in sequential memory access. 

## GCC
#### Normal Use
```
gcc -Wall -c xxx.c              # generate object files 
gcc *.o -o PROGRAM_NAME -lxxx   # link object files together and generate binary program
                                # -lxxx means using library with name libxxx, if a function in the realtime library is used. 
```

#### Flags of C
 - `-D name`        : define at compile time, as 1
 - `-D name=value`  : define at compile time
 - `-I include_path`: additional path to search for include files
 - `-L lib_path`: additional path to search for external linking libraries
 - `-l lib_name`: external linking library names 
 - `-Wall`          : print all warnings during compilation 
 - `-Werror`        : treat all warnings as error
 - `-On`            : optimization flag, with n=0 giving no optimization, and n=1,2,3 giving increasing level of optimization. 
 - `-g`             : adds debug symbols 
 - `-DNDEBUG`       : disable assertion checks 

**Note** that for dynamic linking, `-l` and `-L` only check for library existence and function signature matching. The libraries are required (and searched for) at runtime again. 

For release use** 
`-O3 -DNDEBUG`

**For gdb debugger** 
`-O0 -g`

**For gcov coverage check**
`-fprofile-arcs -ftest-coverage -DNDEBUG -O0`
Note that gcov can only be run after the binary program is run. 


## Using gdb
gdb is an interactive tool used to debug the program (like the Lua debugger). 
`gdb <program_name>`

#### Normal commands: 
`r` or `run`        : run the program until it halts / returns. 
`q` or `quit`       : quit gdb
`p` or `print`      : print the return value of a statement
`bt` or `backtrace` : print the backtrace of the entire stack (of error reporting)
`f` or `frame`      : go to a specific frame (with a frame number following `f`). In a stack trace of error reporting, we are in the outmost frame by default, and local variables in internal frames (e.g. functions) are not visible. 

`info locals`		: show all local variables

`finish`		: finish current function


## Using Valgrind
Valgrind is used for memory debuging, including absence of initialization, memory leak, etc. 
Installation: `sudo apt-get -y install libc6-dbg`
Note that valgrind only detects memory bugs that affect outputs. 

## Using Perf
Perf is a profiler tool to record hardware events
`perf record <program_name> <program_arguments>` record performance events
`perf report` show the report interactively

## Using Cachegrind
Cachegrind is a cache and branch-prediction profiler. 
`valgrind --tool=cachegrind --branch-sim=yes <program_name> <program_arguments>`
The output shows cache misses for instructions (L1), first level data (D1) and last level data cache (LL). It also shows branch prediction misses. 

Use `cg_annotate cachegrind.out.pid` to access details about functions. Adding `xxx.c` to see reports about a specific source file. 

## Make 
Use `-s` to avoid printing commands during `make`

#### Normal Make syntax (in a file with name GNUmakefile, makefile or Makefile)
```make
include filenames   # read another makefile first 

CC := gcc           # simple variable definitions, expand only once (as opposed to recursive expanded variables, defined without the colon) 
CFLAGS := xxx
LDFLAGS := xxx
SOURCES := $(wildcard *.c)  # wildcard expansion not automatic when setting variable or in function
OBJECTS := $(patsubst %.c, %.o, $(SOURCES)) # pattern substitute. When wildcard function is not used, use %.o for pattern

all: PROGRAM_NAME   # conventional name for command `make all`

xxx.o: xxx.c                    # the normal syntax is `target: dependencies`, following by a line of command with a tab upfront. 
    $(CC) $(CFLAGS) -c $<       # the command to be executed

%.o: %.c                        # wildcard expansion automatically sets all .o files
    $(CC) $(CFLAGS) -c $< 

PROGRAM_NAME: xxx.o xxxx.o
    $(CC) $^ -o $@ $(LDFLAGS)

.PHONY: clean                   # add commands as dependencies of .PHONY to avoid confusion with a file with the same name. 
clean:                          # this is what is executed after typing `make clean`
    rm -f PROGRAM_NAME *.o  
```

#### Special variables 
`$@`: target of the rule (the name before colon)
`$<`: the name of the first prerequisite
`$^`: name of all prerequisites, with no repeatition
`$?`: name of all prerequisites newer than the target 












