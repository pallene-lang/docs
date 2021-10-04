## Expressions and Statements

Pallene uses the same set of operators and control-flow statements as Lua.
The only difference is that the type system is more restrictive:

* The logic operators (`not`, `and`, `or`) only operate on expressions of type `boolean` or of type `any`
* The condition of `if`, `while` and `repeat` must be of type `boolean` or of type `any`
* Relational operators (`==`, `<`, etc) must receive two arguments of the same type.
* The arithmetic and concatenation operators don't automatically coerce between numbers and strings.
