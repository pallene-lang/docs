# Overview


Pallene is a statically-typed companion language to Lua.
Pallene functions can be called from Lua and Pallene can call Lua functions as well.

At a first glance, Pallene code looks similar to Lua, but with type annotations.
As a general principle, if a Pallene program runs without errors, then erasing the types and running it as Lua should produce the same result.
The exception are the times when Pallene produces an error, either a compile-time error or a run-time type error.

Here is an example Pallene subroutine for summing the elements in an array of floating-point numbers.
Note that type annotations are required for function argument and return types but for local variable declarations Pallene can often infer the types.

```lua
local m = {}
function m.sum_floats(xs: {float}): float
    local r = 0.0
    for i = 1, #xs do
        r = r + xs[i]
    end
    return r
end
return m
```

If we put this subroutine in a file named `sum.pln` and pass this file to the `pallenec` compiler, it will output a `sum.so` Lua extension module:

```bash
$ ./pallenec sum.pln
```

The `sum.so` file can be loaded from within Lua with `require`, as usual:

```bash
$ ./vm/src/lua
> sum = require "sum"
> print(sum.sum_floats({10.0, 20.0, 30.0})) --> 60.0
```

In this example, we invoke our bundled Lua interpreter because Pallene is only compatible with a specific release version of the interpreter.
The Lua installed in the system might be from an incompatible version.

