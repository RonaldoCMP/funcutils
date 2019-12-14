# OpenSCAD Functional Programming Utilities

This library is a collection of [OpenSCAD](http://www.openscad.org) functions for use with the **function-literals** *experimental* feature.

It is meant to provide algorithms and tools to help build efficient scripts using functional programming techniques in OpenSCAD.

:warning: - Only OpenSCAD **[Development Snapshots](http://www.openscad.org/downloads.html#snapshots)** or **Nightly** builds come with experimental features enabled, and must be newer than **Oct. 23 2019** when **function-literals** was merged into OpenSCAD master.

:warning: - Additionally, the feature must be enabled within OpenSCAD by checking the option under **Preferences -> Features -> function-literals**

:warning: - If building [OpenSCAD from source](https://github.com/openscad/openscad#building-openscad), the experimental features must be enabled during build configuration:
   - Using qmake (primary method): `qmake CONFIG+=experimental [...]`
   - Using cmake (alternative): `cmake -DEXPERIMENTAL=ON [...]`


## General Usage

In order to pass functions as values, function literals are declared slightly differently than "traditional" OpenSCAD functions, so that they exist as an *assignment* in OpenSCAD's variable namespace.

 - function literal: `foo = function(x) x + 1337;`
 - vs traditional function: `function foo(x) = x + 1337;`

:warning: Important note for including files which define function literals: 
 - function literal declarations **_are_ variable assignments**, and `use <filename>` **does not evaluate assignments**.
 - Therefore `include <filename>` **should be used instead**, for files from this library.

Currently OpenSCAD's builtin functions are not available directly as literals (this might change in future), but can be wrapped in anonymous functions for passing into higher-order functions.

Example:
```
echo(filter([1,[],2,3,4,5,undef,"no"];, function(x) is_num(x))); // result: [1,2,3,4,5]
```

## API Reference 

### Library Files
 - <a href="#funcutils">funcutils.scad</a>
   - Main file to include *all* other files in this library
 - <a href="#func">func.scad</a>
   - Minimal collection of common higher-order functions: `map`, `filter`, `fold`
 - <a href="#ops">ops.scad</a>
   - Elementary function literals for comparison, arithmetic and boolean value operations.
 - <a href="#std">std.scad</a>
   - Functions modeled after [C++ STL Algorithms](https://en.cppreference.com/w/cpp/algorithm) mostly operating on ranges of sequences (lists/vectors or strings)
 - <a href="#list">list.scad</a>
   - Various list-based functions which are *not* directly modeled after C++ STL algorithms.
 - <a href="#string">string.scad</a>
   - String specific functions
 - <a href="#types">types.scad</a>
   - Extra type checking functions for which no OpenSCAD buitin exists.

## Dependency / Include Tree

:warning: - Individual library files already `include <...>` their own dependencies.
To avoid duplicate definitions, do not manually include any more-deeply-nested dependencies than the outermost one(s) you choose.
So, if including top-level `functils.scad`, then do not include *any* others.

 - [funcutils.scad](#funcutils)
   - [list.scad](#list)
     - [types.scad](#types)
     - [std.scad](#std)
       - [ops.scad](#ops)
   - [func.scad](#func)
   - [string.scad](#string)

<a name="funcutils"></a>
# funcutils.scad
**Top level file, existing only to include all others.  No function definitions in itself.**

<a name="ops"></a>
# ops.scad
**Elementary Operators**

###  Comparators
 - `eq (x, y)`
   - Equal
 - `ne (x, y)`
   - Not Equal
 - `lt (x, y)`
   - Less Than
 - `gt (x, y)`
   - Greater Than
 - `le (x, y)`
   - Less than or Equal
 - `ge (x, y)`
   - Greater than or Equal

### Arithmetic unary ops
 - `ident (x)`
   - Identity function (returns x unchanged)
 - `neg (x)`
   - Negation

### Arithmetic binary ops
 - `add (x, y)`
   - Addition
 - `sub (x, y)`
   - Subtraction
 - `mul (x, y)`
   - Multiplication
 - `div (x, y)`
   - Division
 - `rem (x, y)`
   - Remainder (`x % y`)
 - <a name="mod"></a>`mod (x, y)`
   - Modulo operator.  Unlike remainder, always returns non-negative value in range: `[0,y)`

### Boolean unary op
 - `not (x)`
   - NOT

### Boolean binary ops
 - `and (x, y)`
   - AND
 - `or (x, y)`
   - OR
 - `xor (x, y)`
   - XOR
 - `nand (x, y)`
   - NAND
 - `nor (x, y)`
   - NOR
 - `xnor (x, y)`
   - XNOR

<a name="list"></a>
# list.scad
**Various list-based functions which are *not* directly modeled after C++ STL algorithms.**

 - `clamp_range (v, begin, step, end)`
   - Return a range with `begin` (inclusive) and `end` (exclusive) clamped between `0` and `len(v)`.
   - Allows negative values to wrap but *only once* (`-1` becomes index of last element, and values less than `-len(v)` clamp to `0`).
   - mainly for use by [slice](#slice) function
 - `def (x, default)`
   - return `x`, or `default` if `x` undefined
 - `get (v, i)`
   - get element from `v` with wrapping, using index [mod](#mod)(i,len(v))
 - `insert (v, x, i)`
   - insert element `x` into `v` at index `i`
 - `insertv (v, i, xs)`
   - insert vector of elements `xs` into `v` at index `i` 
 - `insert_sorted (v, x, cmp=lt)`
   - insert `x` into sorted vector `v`
 - `insertv_sorted (v, xs, cmp=lt)`
   - insert vector `xs` into sorted vector `v`
   - `xs` does not need to be sorted
```
echo(insertv_sorted([1,2,3,5,5,8,12],[6,4,7,-8,4,3,5,-6,5,3,2,1,10,0,-1]));
echo(insertv_sorted([],[for (x=rands(0,10000,10000)) floor(x)]));
```
 - `replace_range (v, range, xs)`
   - replace elements of `v` in range with elements from vector `xs`
   - `range` is vec2 containing `[begin,end)` indices.
   - `xs` does not need to be same length as range
 - `rotl (v, l=1)`
   - Rotate elements in vector `v`, by `l` positions to the left.
 - `rotr (v, l=1)`
   - Rotate elements in vector `v`, by `l` positions to the right.
 - `sublist (s, b=0, e)`
   - Extract sub-list given `begin` (inclusive) and `end` (exclusive). If `end` not specified, go to end of list
 - <a name="slice"></a>`slice (v, begin, step, end, range)`
   - slice vector `v`, similar to Python's builtin [slice](https://docs.python.org/3.3/library/functions.html#slice) function or [extended indexing syntax](https://docs.python.org/3.3/library/stdtypes.html#common-sequence-operations).
   - all params after `v` are optional
   - `range` param overrides others
   -  unnamed range may be optionally positioned in place of `begin`


<a name="std"></a>
# std.scad
**Functions modeled after [C++ STL Algorithms](https://en.cppreference.com/w/cpp/algorithm) (incomplete, see [stl algorithm status](stl_algorithm_status.md) for details)**

### Non-modifying sequence operations 

 - `end (v,last)`  (ref: [std::end](https://en.cppreference.com/w/cpp/iterator/end), loosely based)
   - return `last`, OR length of `v` if `last` not defined.
 - `find_if (v, first=0, last, p)` (ref: [std::find](https://en.cppreference.com/w/cpp/algorithm/find))
  - return index of first element in range `[first,last)` which matches predicate `p`, or `last` if none found.
 - `find_if_not (v, first=0, last, p)` (ref: [std::find](https://en.cppreference.com/w/cpp/algorithm/find))
  - return index of first element in range `[first,last)` which does *not* match predicate `p`, or `last` if none found.
 - `find (v, value, first=0, last, cmp=eq)` (ref: [std::find](https://en.cppreference.com/w/cpp/algorithm/find))
   - return index of first element in range `[first,last)` equivalent to `value`, or `last` if none found.
```
v=[6,4,7,-8,4,3,5,-6,5,3,2,1,10,0,-1];
echo(find(v,-8));
echo(find(v,-1));
echo(find(v,undef));
echo(find([0],0));
echo(find([0],1));
echo(find([],undef));
```
 - `all_of (v, first=0, last, p)` (ref: [std::all_of](https://en.cppreference.com/w/cpp/algorithm/all_any_none_of))
   - Checks if unary predicate `p` returns `true` for all elements in the range `[first, last)`. 
 - `any_of (v, first=0, last, p)` (ref: [std::any_of](https://en.cppreference.com/w/cpp/algorithm/all_any_none_of))
   - Checks if unary predicate `p` returns `true` for at least one element in the range `[first, last)`. 
 - `none_of (v,first=0,last,p)` (ref: [std::none_of](https://en.cppreference.com/w/cpp/algorithm/all_any_none_of))
   - Checks if unary predicate `p` returns `true` for no elements in the range `[first, last)`. 
```
for(v=[ [], [0,1,2,3], [0,1,2,3,0/0], [[],false,true,"ok"] ]) let(p=function(x)is_num(x)) {
  if (all_of(v,p=p)) echo(str("all_of(",v,",is_num) = true"));
  if (any_of(v,p=p)) echo(str("any_of(",v,",is_num) = true"));
  if (none_of(v,p=p)) echo(str("none_of(",v,",is_num) = true"));
}
```
 - `contains (v, value, first, last, cmp=eq)`
   - No direct analogue in C++ STL, but a variation of `std::find` returning bool, or loosely based on [std::map::contains](https://en.cppreference.com/w/cpp/container/map/contains)
   - return `true` if any element in range `[first,last)` is equivalent to `value`
```
echo(contains([6,4,7,-8,4,3,5,-6,5,3,2,1,10,0,-1],3.14));
echo(contains([6,4,7,-8,4,3,5,-6,5,3,2,1,10,0,-1],-1));
```

### Modifying sequence operations

 - `remove(v,first=0,last)` ref: [std::remove](https://en.cppreference.com/w/cpp/algorithm/remove)
   - remove range of elements [first,last) from list
 - `remove_if(v,first=0,last,p)` ref: [std::remove_if](https://en.cppreference.com/w/cpp/algorithm/remove)
   - remove elements from range which match predicate
```
echo(remove([0,1,2,3,4,5,6,7,8,9],3,7));
echo(remove([0,1,2,3,4,5,6,7,8,9]));
echo(remove_if([0,1,2,3,4,5,6,7,8,9],p=function(x)x%3==0));
```
 - `replace (v, old_value, new_value, first=0, last, cmp=eq)` ref: [std::replace](https://en.cppreference.com/w/cpp/algorithm/replace)
   - Replaces all elements that are equivalent to `old_value` with `new_value`, in the range `[first, last)`.
 - `replace_if (v, new_value, first=0, last, p)` ref: [std::replace_if](https://en.cppreference.com/w/cpp/algorithm/replace)
   - Replaces all elements for which predicate `p` returns `true`, with `new_value`, in the range `[first, last)`.
```
echo(replace([0,1,2,3,1,4,5,6,1,7,8,1,9],1,[]));
echo(replace_if([0,1,2,3,1,4,5,6,1,7,8,1,9],0,p=function(x)x<5));
```
 - `reverse (v, first=0, last)` ref: [std::reverse](https://en.cppreference.com/w/cpp/algorithm/reverse)
   - Reverse a range of elements
```
echo(reverse([0,1,2,3,4,5,6,7,8,9]));
echo(reverse([0,1,2,3,4,5,6,7,8,9],3,7));
```
 - `std_rotate (v, first=0, first_n, last)` ref: [std::rotate](https://en.cppreference.com/w/cpp/algorithm/rotate)
   - Reorders elements in the range [first,last) such that sub-ranges:
     - [first_n, last) moves to the beginning of the range, and
     - [first, first_n) moves to end of the range.
```
// move elements [3,7) to position 12: https://www.youtube.com/watch?v=W2tWOdzgXHA&feature=youtu.be&t=630
echo(std_rotate([0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15],3,7,12)); 
// move elements [8,12) to position 3: https://www.youtube.com/watch?v=W2tWOdzgXHA&feature=youtu.be&t=645
echo(std_rotate([0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15],3,8,12)); 
```
 - `unique (v, first=0, last, cmp=eq)` ref: [std::unique](https://en.cppreference.com/w/cpp/algorithm/unique)
   - Removes all but the first element from every group of consecutive equivalent elements in range `[first,last)`.
   - `v` should be pre-sorted if uniqueness across entire vector is desired.
```
echo(unique(insertv_sorted([],[1,2,3,1,2,3,3,4,5,4,5,6,7])));
```

### Partitioning operations

 - `stable_partition (v, first=0, last, p)` ref: [std::stable_partition](https://en.cppreference.com/w/cpp/algorithm/stable_partition)
   - Reorders the elements in the range `[first, last)` in such a way that all elements for which the predicate `p` returns `true` precede the elements for which predicate `p` returns `false`. 
   - Relative order of the elements is preserved.
```
is_even = function(x)x%2==0;
echo(stable_partition([0,1,2,3,4,5,6,7,8,9],p=is_even));
```

### Sorting operations
 - None yet implemented

### Binary search operations (on sorted ranges)
 - `upper_bound (v, value, first=0, last=undef, cmp=lt)`  ref: [std::upper_bound](https://en.cppreference.com/w/cpp/algorithm/upper_bound)
   - return index of first element in range `[first,last)` which is greater than `value`.
 - `lower_bound (v, value, first=0, last=undef, cmp=lt)` ref: [std::lower_bound](https://en.cppreference.com/w/cpp/algorithm/lower_bound)
   - return index of first element in range `[first,last)` which is *not less* than `value`.
 - `binary_search (v, value, first=0, last=undef, cmp=lt)` ref: [std::binary_search](https://en.cppreference.com/w/cpp/algorithm/binary_search)
   -  return `true` if range `[first,last)` contains `value`.
```
v = [1, 1, 2, 3, 3, 3, 3, 4, 4, 4, 5, 5, 6];
echo(sublist(v,lower_bound(v,4),upper_bound(v,4)));
echo(binary_search([1,2,3,5,5,8,12],3));
```

### Numeric operations

 - `accumulate (v, first=0, last=undef, init=0, op=add)` ref: [std::accumulate](https://en.cppreference.com/w/cpp/algorithm/accumulate)
   - same as "fold" but with optional params for first, last, op (default addition)
```
echo(accumulate([1,2,3,4,5]));
echo(accumulate([1,2,3,4,5],init=1,op=function(x,y)x*y));
```

<a name="string"></a>
# string.scad

**String specific functions**

Strings may be iterated over character by character, and many of the same functions/algorithsm that work on vectors will also "work" on strings.
However the value returned is typically a vector of single-character strings, when what is expected is a single string.
This is why string-specific functions have their own place here.

- `substr (s, b, e)`
   - extract substring given begin(inclusive) and end(exclusive)
   - if end not specified, go to end of string 
 - `join (strs, delimiter="")`
   - converts a vector of values into a single string, with optional delimiter between elements
   - uses efficient binary tree based join, where depth of recursion is `log_2(len(strs))`.
 - `fixed (x, w, p, sp="0")`
   - format number `x` as string with fixed width `w` and precision `p`.

<a name="types"></a>
# types.scad
**Extra type checking functions for which no OpenSCAD buitin exists.**

 - `is_range (x)`
   - return true only if x is a range object
 - `is_nan (x)`
   - return true only if x is the [nan](https://en.wikibooks.org/wiki/OpenSCAD_User_Manual/General#Numbers) value
 - `has_nan (x)`
   - return true only if x is a list containing [nan](https://en.wikibooks.org/wiki/OpenSCAD_User_Manual/General#Numbers), or is itself [nan](https://en.wikibooks.org/wiki/OpenSCAD_User_Manual/General#Numbers)
 - `has_function (v)`
   - return true only if x is a list containing a function
```
for (
  x=[undef,1/0,-1/0,0/0,[],[1,2,3],[1,0/0,2],[1,function(x)x,2],[1,function(x)x,0/0,2],
    "hi!","",function(x)x,true,false,0,1,-1,[0:-10],[0:1:10],[0:0:0],[0:1:2]]
  ) {
  if (is_nan(x)) echo(str("is_nan(",x,") = true"));
  if (has_nan(x)) echo(str("has_nan(",x,") = true"));
  if (has_function(x)) echo(str("has_function(",x,") = true"));
  if (is_range(x)) echo(str("is_range(",x,") = true"));
}
```
