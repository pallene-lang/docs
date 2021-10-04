## The Pallene Type System

Pallene's type system includes the usual Lua primitive types (`nil`, `boolean`, `float` and `integer`), as well as strings, arrays, tables, functions, and records.
There is also a catch-all type `any`, which can refer to any Lua or Pallene value.

### Contents

- [Primitive types](#primitive-types)
- [Integers and floats](#integers-and-floats)
- [Strings](#strings)
- [Arrays](#arrays)
- [Tables](#tables)
- [Functions](#functions)
- [Loops](#loops)
- [Records](#records)
- [Any](#any)

### Primitive types

Pallene's primitive types are similar to Lua's:

- `nil`
- `boolean`
- `integer`
- `float`

### Integers and floats

In Pallene there are separate types for `integer` and `float` instead of a single `number` type.
There are no automatic coercions between the two types, meaning that code such as the following is a type error in Pallene:

```lua
local x: float = 0 -- Type error. Should use 0.0 instead.
```

That said, most arithmetic operators still work if one of the parameters is an integer and the other is a float.
If you just want to convert an integer to float, the recommended idiom is to multiply it by 1.0.

```lua
local i: integer = 10
local x: float = i + 3.4
local y: float = i * 1.0
```

### Strings

Pallene also has a `string` type, for Lua strings.
The syntax for string literals is the same as in Lua
You can use single quotes, double quotes, or `[[`.

The primitive operators that operate on strings are the concatenation operator `..`,  the length operator `#`, and the comparison operators (`==`, `~=`, `<`, `>`, `<=`, `>=`).

Pallene also implements some functions from the `string` library.
Currently we implement `string.char` and `string.sub`  but more may be implemented in the future.

### Arrays

Array types in Pallene are written `{ t }`, where `t` is any Pallene type. For example, `{ { integer } }` is the type for an array of arrays of integers.

Pallene arrays are implemented as Lua tables, and Pallene also uses the same syntax for array creation:

```lua
local xs: {integer} = {10, 20, 30}
```

Like Lua, reading from an "out of bounds" index produces `nil`, which results in a run-time type error unless the type of the array elements is `any`.
Notice that Pallene doesn't accept arrays of nil.

One important thing to know about array literals in Pallene is that they must be accompanied by a type annotation.
Pallene cannot infer their type otherwise because expressions like the empty list `{}` don't have an obvious best type.

```lua
-- This produces a compile-time error
-- "missing type hint for array or record initializer"
local xs = {1.0, 2.0, 3.0}
```

Nevertheless, Pallene is still able to infer then type of an array literal if they appear as an argument to a function, or in another position in the program that has a known expected type.

```lua
local result = sum_floats({1.0, 2.0, 3.0})
```

### Tables

Table types in Pallene are writen as `{ field: t [, field2: t2, ...] }`, where `field` is an identifier and `t` is any Pallene type.
For instance, `{ x: integer, y: integer }` is the type for a table with the fields `x` and `y` that are integers.

Like arrays, Pallene tables are implemented as Lua tables and Pallene uses the same Lua syntax for their creation:

```lua
typealias point = {x: integer, y: integer}
local p: point = {x = 10, y = 20}
```

Notice that all fields must be initialized in the Pallene initalizer list, even fields with the type `any` which could be `nil`.
Tables that come from Lua may have absent fields; like Lua, absent fields are considered to be nil.

It is possible to get and set fields in tables using the usual dot syntax:

```lua
p.x = 30
print(p.x) --> 30
```

In the current version of Pallene, the length of the field should be at max `LUAI_MAXSHORTLEN` characters.
In Lua 5.4, the default value for this constant is 40 characters.

### Functions

Function types in Pallene are created with the `->` type constructor.
For example, `(a, b) -> (c)` is the function type for a function that receives two arguments (the first of type `a` and the second of type `b`) and returns a single value of type `c`.
If the function receives a single input parameter, or returns a single value, the parenthesis can be omitted.
The following are more examples of valid function types:

```lua
int -> float
(int, int) -> float
(int, int) -> (float, float)
string -> ()
```

The arrow type constructor is right-associative.
That is, `a -> b -> c` means `a -> (b -> c)`.

A Pallene variable of function type may refer to either a statically-typed Pallene function or to a dynamically typed Lua function.
When calling a dynamically-typed Lua function from Pallene, Pallene will check whether the Lua function returned the correct types and number of arguments and it will raise a run-time error if it does not receive what it expected.

Note that lambda functions cannot be explicitly type annotated.
They're always checked against a type annotation somewhere else in the program, usually a function parameter or variable
declaration.

```lua
local twice: integer -> integer = function (x) -- note how `x` isn't annotated
	return x * 2
end
```

### Loops

Pallene supports the usual variety of loops seen in Lua.
Pallene's `for-in` loops carry some semantic importance that might be of relevance.
An example of such a loop might look like this:

```lua
local function sum_list(xs: {integer}) do
    local sum: integer = 0
    for _, x: integer in ipairs(xs) do
        sum = sum + x
    end
end
```

The variables on the left hand side of a loop (here `_` and`x`) may optionally
be type annotated.
If no annotation is found, their type is assumed to be `any`.

The right hand side of a for-in loop following "`in`" expects 3 values (or
a call returning 3 values, like `ipairs(xs)`).

The first is the iterator function which accepts 2 arguments, both of type `any`, and returns as many values of type `any` as found in the left hand side of the loop. In this case, the iterator function has type `(any, any) -> (any, any)`.

The second is the `state` variable of type `any` which is the table that the loop is iterating over.

The third and last value is the `control` variable of type `any`.
Under most contexts, it is the current index or slot of the table under inspection.

As an example, consider the same `sum_list` function from above written without `ipairs` using a hand written iterator function:

```lua
local function iter(arr: {any}, prev: integer): (any, any)
    local i = prev + 1
    local x = arr[i]
    if x == (nil as any) then
        return nil, nil
    end

    return i, x
end

typealias iterfn = (any, any) -> (any, any)
local function my_ipairs(xs: {any}): (iterfn, any, any)
    return iter, xs, 0
end

local function sum_list(xs: {integer}): integer
    local sum = 0
    for _, x: integer in my_ipairs(xs) do
        sum = sum + x
    end
end
```

### Records

Record types in Pallene are nominal and should be declared in the top level.
The following example declares a record `Point` with fields `x` and `y` which are floats.

```lua
record Point
    x: float
    y: float
end
```

These points are created and used with a similar syntax to Lua:

```lua
local p: Point = {x = 10.0, y = 20.0}
local r2 = p.x * p.x + p.y * p.y
```

Pallene records are implemented as userdata, and are *not* Lua tables.
You cannot create a Lua table with an `x` and `y` field and pass it to a Pallene function expecting a Point.
The fields of a Pallene record can be directly accessed by Pallene functions using dot notation but are *cannot* be accessed by Lua functions the same way.
From the point of view of Lua, Pallene records are opaque.
If you want to allow Lua to read or write to a field, you shold export appropriate getter and setter functions.

### Any

Variables of type `any` can store any Lua or Pallene value.

```lua
local x: any = 10
x = "hello"
```

Similarly, arrays of `any` can store anys of varied types

```lua
local xs: {any} = {10, "hello", 3.14}
```

Upcasting a Pallene value to the `any` type always succeeds.
Pallene also allows you to downcast from `any` to other types.
This is checked at run-time, and may result in a run-time type error.

```lua
local v = (17 as any)
local s = (v as string)  -- run-time error: v is not a string
```

The `any` type allows for a limited form of dynamic typing.
The main difference compared to Lua is that Pallene does not allow you to perform any operations on a `any`.
You may pass a `any` to a functions and you may store it in an array but you cannot call it, index it, or use it in an arithmetic operation:

```lua
local function f(x: any, y: any): any
    return x + y -- compile-time type error: Cannot add two anys
end
```

You must first downcast the `any` to the appropriate type.
Sometimes the Pallene compiler can do this automatically for you but in other situations you may need to use an explicit type annotation.
The reason for this is that, for performance, Pallene must know at compile-time what version of the arithmetic operator to use.

```lua
local function f(x: any, y: any): integer
    return (x as integer) + (y as integer)
end
```