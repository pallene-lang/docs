## Type annotations and type inference

Pallene is a statically-typed language, which means that every variable and expression has a known type, determined at compilation time.
Sometimes this may be the catch-all type `any`, but it is still known at compilation time.
Similarly to most other statically-typed languages, Pallene allows you to add type annotations to variables, functions, and expressions.
(This is one of the few syntactical differences between Lua and Pallene.)
Pallene type annotations for variables and functions are written using colons.
For expressions the colon is already used for method calls, so Pallene uses the `as` operator instead.

```lua
function foo(x : any) : integer
   local y: integer = (x as integer)
   return y + y
end
```

Unlike languages like C or Java, Pallene does not require type annotations on every variable.
It uses a bidirectional type-checking system that is able to infer the types of almost all variables and expressions.
Roughly speaking, you must include type annotations for the parameters and return types of top-level functions, and almost everything else can be inferred from that.
For example, notice how the `sum_floats` from the Brief Overview section does not include a type annotation for the `result` and `i` variables.

If a local variable declaration doesn't have an initializer, it must have a type annotation:
```lua
function contrived(): integer
    local x:integer
    x = 10
    return x
end
```

### Automatic type coercions

In some places in a Pallene program there is a natural "expected type".
For example, the type of a parameter being passed to a function is expected to be the type described by the corresponding function type.
Similarly, there is also an expected type for expressions surrounded by a type annotation, or values being assigned to a variable of known type.

If the expected type of an expression is `any` but the inferred type is something else, Pallene will automatically insert an upcast to `any`.
Similarly, if the inferred type is `any` but the expected type is something else, Pallene will insert a downcast from `any`.
For instance, one of the code examples from the Any section of this manual can be rewritten to use automatic coercions as follows:

```lua
local v: any  = 17
local s: string = v
```

In addition to allowing conversions to and from `any`, Pallene also makes implicit conversions to and from types that contain `any` in compatible ways.
For example, `{ any }` and `{ integer }` are considered to be compatible, and one may be used where the other is expected.
Similar, for function types `integer -> integer`, `any -> integer`, `integer -> any`, and `any -> any` are all compatible with each other.
These automatic coercions between array and function types never fail at run-time.

To illustrate this, consider the following function for inserting an element in a list.

```lua
function insert(xs: {any}, v:any)
    xs[#xs + 1] = v
end
```

Since the parameter to the insert function is an array of `any`, we can use it to add elements to lists of any type:

```lua
local ns: {integer} = {10, 20, 30}
insert(ns, 40)

local ss: {string} = {"hello"}
insert(ss, "world")
```

However, the insert function only guarantees that its first parameter is an array.
If the input is an homogeneous array, the insert function does not ensure that the value being inserted has the same type.
If a value of the "wrong" type is inserted, this will only be noticed when attempting to read from the array.

```lua
local ns: {integer} = {10, 20, 30}
insert(ns, "boom!")
local x1 : integer = ns[1]
local x2 : integer = ns[4] -- run-time error
```