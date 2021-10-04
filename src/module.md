## Structure of a Pallene module

Lua allows many idiomatic ways of defining a module.
Pallene programs, however, must follow a stricter style.
A Pallene module must start with a local variable declaring the module name and it must finish with a return statement returning that variable.
The type annotation for the module variable is optional, but if present it should say "module".
The body of the module consists of a sequence of type declarations, module-local variables, and function definitions.

```lua
local m: module = {}
-- ...
return m
```

### Type aliases

Creates an alias for a previously-declared type with the following syntax:

```lua
typealias <name> = <type>
```

Type alias in Pallene currently cannot be used to declare recursive types, such as:

```lua
typealias T = {T}
```

### Record declarations

Record declarations consist solely of record declarations.

```
record <name>
    <name> : type
    ...
end
```

### Module-local variables

Module-local variables are declared with the following syntax:

```
local <name> [: type] {, <name> [: type]} = <exp> {, <exp>}
```

It is possible to declare multiple variables at once. The behaviour for expressions that are function calls is the same as in Lua.

### Functions

Functions are declared as follows:

```
[local] function <name>([<params>])[: <rettypes>]
    <body>
end
```

A `local` function is only visible inside the module it is defined.
To export a function, have it's name be a field of the module table, e.g. `function m.foo()`.

Functions that are created using a function statement are immutable in Pallene; you may not re-assign them to a different function.
It is a compile-time error to declare two exported functions with the same name.
The return types `<rettypes>` are optional; if not given it is assumed that the function does not return anything.
If the function returns more than one value, you must use parenthesis around the return types.

Parameters are a comma-separated list of `<name>: <type>`.
Two parameters cannot have the same name.

Pallene functions can be recursive.
Blocks of mutually-recursive functions are also allowed, as long as the mutually-recursive functions are declared next to each other, without any type or variable declarations between them.

```lua
function m.f()
    m.g() -- ok
end
function m.g()
    m.h() -- not allowed
end
local _ = 17
function m.h()
    m.f() -- ok
end
```

To define a block of mutually-recursive local functions you can use a forward declaration.
The forward declaration must be adjacent to the functions.
The functions must also use the function statement syntax, instead of assignment statements.

```lua
local odd, even
function odd(n:integer): boolean
    if n == 0 then return false else return even(n-1) end
end
function even(n:integer): boolean
    if n == 0 then return true else return odd(n-1) end
end
```