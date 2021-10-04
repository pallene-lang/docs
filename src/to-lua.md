
## Pallene to Lua translator

There are situations where removal of type annotations are useful.
 * The main premise of Pallene is that it is compatible with Lua.
   This compatibility is defined as removing type annotations from a Pallene program results in transforming it into a Lua program.
   A Pallene to Lua translator will allow us to check whether this property is still valid.
 * Provides greater portability, interoperability and integration with existing Lua codebase and tools.
 * The Pallene developers could check if the unit tests are obeying the "gradual guarantee".
 * In the benchmarks that do not use LuaJIT features, we could generate the Lua version of the code from the Pallene version.

With the help of the Pallene to Lua translator, users can remove Pallene type annotations to generate plain Lua.
You can run the compiler with the flag `--emit-lua` flag to cause the compiler to generate plain Lua instead of C.

Consider the following example written in Pallene.
```lua
function sum(values: { float }): float
    local s: float = 0.0
    for i = 1, #values do
        s = s + values[i]
    end
    return s
end
```

For the above example, the Pallene to Lua translator produces the following Lua program.
```lua
function sum(values)
    local s = 0.0
    for i = 1, #values do
        s = s + values[i]
    end
    return s
end
```

## The Complete Syntax of Pallene

Here is the complete syntax of Pallene in extended BNF.
As usual, {A} means 0 or more As, and \[A\] means an optional A.

```
    program ::= {toplevelrecord | topleveltypealias | toplevelvar | toplevelfunc | ';'}

    toplevelrecord ::= record Name {recordfield} end
    recordfield ::= NAME ':' type [';']

    topleveltypealias ::= typealias NAME = type

    toplevelvar ::= local|export NAME [':' type] {',' NAME [':' type]} '=' explist

    toplevelfunc ::= local|export function NAME '(' [paramlist] ')'  [':' typelist ] block end

    paramlist ::= NAME ':' type {',' NAME ':' type}

    type ::= nil | integer | float | boolean | string | any
             | NAME
             | '{' type '}'
             | '{' [tabletypefields] }'
             | typelist '->' typelist

    tabletypefields ::= NAME ':' type { ',' NAME ':' type}

    typelist ::= type | '(' [type, {',' type}] ')'

    block ::= {statement} [returnstat]

    statement ::=  ';' |
        varlist '=' explist |
        funccall |
        do block end |
        while exp do block end |
        repeat block until exp |
        break |
        if exp then block {elseif exp then block} [else block] end |
        forstat |
        local name [':' type] '=' exp

    forstat ::= for NAME [':' type] '=' exp ',' exp [',' exp] do block end |
        for NAME [':' type] {',' NAME [':' type]} in explist do block end

    returnstat ::= return exp [';']

    var ::=  NAME | exp '[' exp ']' | exp '.' Name

    exp ::= nil | false | true | NUMBER | STRING | initlist | exp as type
        | unop exp | exp binop exp | '(' exp ')'
        | NAME | exp '[' exp ']' | exp '.' NAME | funccall

    varlist ::= var {',' var}
    explist ::= exp {',' exp}

    funccall ::= exp funcargs

    funcargs ::= '(' [explist] ')' | initlist | STRING

    initlist ::= '{' fieldlist '}'
    fieldlist ::= [ field {fieldsep field} [fieldsep] ]
    field ::= exp | NAME '=' exp
    fieldsep ::= ',' | ';'

    unop ::= '-' | not | '#' | '~'

    binop ::=  '+' | '-' | '*' | '/' | '//' | '^' | '%' |
        '&' | '~' | '|' | '>>' | '<<' | '..' |
        '<' | '<=' | '>' | '>=' | '==' | '~=' |
        and | or
```