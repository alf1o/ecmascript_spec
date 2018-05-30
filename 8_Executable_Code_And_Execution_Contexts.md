# Lexical Environments

A `Lexical Environment` is a specification type and is used to associate identifiers to variables and functions based on the lexical nesting structure of the ES code.

Given an identifier name, the `Lexical Environment` in which we are in determines which variable or function we can access with that name.

`Lexical Environment`s are associated with specific syntactic structures like function declarations, block statements, catch clause of a try/catch block.
Every time such code is evaluated a new `Lexical Environment` is created.

A `Lexical Environment` consists of an `Environment Record` and a reference to an outer `Lexical Environment`.

An `Environment Record` keeps track of the identifiers defined within the scope associated with its `Lexical Environment`

The outer environment reference is used to model the nesting structure of `Lexical Environment`s.
The reference of a `Lexical Environment` is another `Lexical Environment` that surrounds it.
The same `Lexical Environment` can be the reference of many inner `Lexical Environment`s.

A *global environment* is a `Lexical Environment` with a reference of **null**, meaning there is no outer `Lexical Environment` containing it. The global environment can be prepopulated with bindings and includes an associated global object. The properties of the global object provide some of the global environment bindings. As ES code executes, additional properties can be added to the global object and the initial properties can be modified.

A *module environment* is a `Lexical Environment` that contains the bindings of the top-level declarations of a module and also the bindings imported inside this module. Its outer environment is the global environment.

A *function environment* is a `Lexical Environment` that corresponds to the invocation of a function. A function environment has its own `this` binding and `super` binding.

`Lexical Environment`s and `Environment Record`s are specification values and they can't be directly accessed by an ES program.

## Environment Records

An `Environment Record` keeps track of variable and function names and their associated values within a certain `Lexical Environment`.

There are different kinds of `Environment Record`s: *declarative `Environment Record`s*, *object `Environment Record`s*, *global `Environment Record`s*, *function `Environment Record`s* and *module `Environment Record`s*.

Declarative `Environment Record`s are created by syntactic constructs that directly associated bindings with ES language values (i.e. function declarations, variable declarations, catch clauses).

Object `Environment Record`s are created by syntactic constructs that associate bindings with the properties of an object (i.e. `with`). An object `Environment Record` is associated with an object called its binding object. An object `Environment Record` binds identifiers names that correspond to the property names of its binding object.

`Environment Record`s can be thought about in a class-oriented way where `Environment Record` is an abstract class which has 3 subclasses: declarative, object and global `Environment Record`s.
Function and module `Environment Record`s are subclasses of declarative `Environment Record`s.

- `Environment Record`
  - object `Environment Record`
  - global `Environment Record`
  - declarative `Environment Record`
    - function `Environment Record`
    - module `Environment Record`

The `Environment Record` "class" provides specification methods that have a different algorithm on each "subclass".

- `HasBinding(N) -> boolean`: determine if the `Environment Record` has a binding for the `string` value `N`.

- `CreateMutableBinding(N, D)`: create a new, non-initialized mutable binding in the `Environment Record`. The `string` value `N` is the name of the identifier. `D` is a `boolean` and if it is **true** then this binding can be deleted.

- `CreateImmutableBinding(N, S)`: create a new, non-initialized immutable binding. The `string` value `N` is the identifier name. `S` is a `boolean` and if it is **true** then attempts to assign a new value to this binding will throw an exception.

- `InitializeBinding(N, V)`: set the value of an existing but non-initialized binding with the name of `N`. `V` is the value to be assigned and can be any ES language value.

- `SetMutableBinding(N, V, S)`: set the value of an existing mutable binding with the name of `N`. `V` is the value to be assigned. `S` is a `boolean`. If it is **true** and the binding `N` cannot be set, then throw a `TypeError`.

- `GetBindingValue(N, S) -> any`: return the value of an existing binding with the name of `N`. `S` is a `boolean` and if it is **true** and the binding does not exist throw a `ReferenceError`. If the binding exists but is not initialized then a `ReferenceError` is always thrown.

- `DeleteBinding(N) -> boolean`: delete a binding with the name `N`. If `N` is successfully deleted or if it doesn't exists, then return **true**. If `N` can't be deleted, return **false**.

- `HasThisBinding() -> boolean`: return **true** if the `Environment Record` has a `this` binding. Else **false**.

- `HasSuperBinding() -> boolean`: return **true** if the `Environment Record` has a `super` binding. Else **false**.

- `WithBaseObject() -> object|undefined`: if the `Environment Record` is associated with a `with` object, then return that object. Else return **undefined**.

### Declarative Environment Records

Each declarative `Environment Record` is associated with an ES program scope containing variable, constant, let, class, module, import and/or function declarations. A declarative `Environment Record` binds the identifiers defined by the declarations contained within its scope.

### Object Environment Records

An object `Environment Record` is associated with an object called its binding object. An object `Environment Record` binds the identifier names that directly correspond to the property names of its binding object. Property keys that are not `string` values are not included in the object `Environment Record`. Both own and inherited properties are included regardless of their enumerability.

Properties can be added and removed from an object, this means that the bindings of an object `Environment Record` can also change as side-effects of such operations. These bindings are always considered mutable bindings, even if the associated properties are non-writable.
Basically, immutable bindings don't exists for an object `Environment Record`.

### Function Environment Records

A function `Environment Record` is a declarative `Environment Record` used to represent the scope of a function and if the function is not an arrow function, this `Environment Record` also provides a `this` binding.

### Global Environment Record

A global `Environment Record` is used to represent the outmost scope of an ES program. It provides the bindings for the built-in globals, the properties of the global object and all the top-level declarations.

A global `Environment Record` is a single `Record`, but it actually is the result of the encapsulation of both an object `Environment Record` and a declarative `Environment Record`.

The object `Environment Record` has the global object as its binding object. It contains the bindings of all the built-in globals, and also all the bindings created by function declarations and variable declarations with the `var` keyword.
Bindings on the object `Environment Record` can be created both explicitly by declarations or implicitly by adding properties on the global object. To distinguish between them, the global `Environment Record` keeps a list of all the identifier names created explicitly.

The bindings for the other declarations (i.e. `let` and `const`) are contained in the declarative `Environment Record`.

### Module Environment Records

A module `Environment Record` is a declarative `Environment Record` used to represent the scope of a module. In addition to mutable and immutable bindings a module `Environment Record` also provides immutable import bindings that provide indirect access to a binding that exists in a different `Environment Record`.

## Operations on Lexical Environments

### GetIdentifierReference(*lex, name, strict*)

This abstract operation takes in a `Lexical Environment` called `lex` which can be **null**, a `name` which is a `string` value and a `boolean` called `strict`.

It specifies the process of scope look-up for a certain identifier name and returns a `Reference`, which might be **undefined** if the binding can't be found.

The `GetIdentifierReference` abstract operation:
- If `lex` is **null** create and return a `Reference` value with a base value component **undefined** , a referenced name component of `name` and a strict reference flag of `strict`
- Else retrieve the `Environment Record` of `lex`
- Call `HasBinding(name)` on this envRec
- If the result is **true** create and return a `Reference` value with base value component envRec, a referenced name component of `name` and a strict reference flag of `strict`
- Else retrieve the value of the `lex` environment reference
- Return the result of calling `GetIdentifierReference` passing in that value, `name` and `strict`

### NewDeclarativeEnvironment(*E*)

This abstract operation takes in a `Lexical Environment` called `E` and creates a new `Lexical Environment` whose environment reference is `E`.

The `NewDeclarativeEnvironment` abstract operation:
- Create a new `Lexical Environment` env
- Create a new empty declarative `Environment Record` envRec
- Set envRec to be the `Environment Record` of env
- Set the environment reference of env to be `E`
- Return `E`

### NewObjectEnvironment(*O, E*)

This abstract operation creates a new `Lexical Environment` with an object `Environment Record`.

The `NewObjectEnvironment` abstract operation:
- Create a new `Lexical Environment` env
- Create a new object `Environment Record` envRec with binding object `O`
- Set envRec to be the `Environment Record` of env
- Set the environment reference of env to be `E`
- Return env

### NewFunctionEnvironment(*F, newTarget*)

This abstract operation is called with a function object `F`, which is the function that caused the creation of this `Lexical Environment`, and a `newTarget` argument which is either an object or **undefined** and creates a new `Lexical Environment`, with a function `Environment Record`, for `F`.

The `NewFunctionEnvironment` abstract operation:
- Create a new `Lexical Environment` env
- Create a new, empty, function `Environment Record` envRec
- Set `envRec.[[FunctionObject]]` to be `F`
- If `F` is an arrow function (its `[[ThisMode]]` internal slot is `lexical`) then set `envRec.[[ThisBindingStatus]]` to be `"lexical"`
- Else set it to be `"uninitialized"`
- Set `envRec.[[HomeObject]]` to be the value of `F.[[HomeObject]]` (is not **undefined** only if the function `F` has a `super` property)
- Set `envRec.[[NewTarget]]` to be `newTarget` (is not **undefined** only if this `Lexical Environment` was created by a `[[Construct]]` call)
- Set envRec to be the `Environment Record` of env
- Set the environment reference of env to be `F.[[Environment]]`
- Return env

### NewGlobalEnvironment(*G, thisValue*)

This abstract operation creates a new global `Lexical Environment`. A global `Lexical Environment` has an `Environment Record` which is both an object `Environment Record` and a declarative `Environment Record` and has a **null** environment reference.

The argument `G` is the global object and is used as binding object of the object `Environment Record`.
The argument `thisValue` is the value for the `this` keyword in case the default binding rule applies.

The `NewGlobalEnvironment` abstract operation:
- Create a new `Lexical Environment` env
- Create a new object `Environment Record` objRec with `G` as binding object
- Create a new, empty, declarative `Environment Record` dclRec
- Create a new global `Environment Record` globalRec
- Set `globalRec.[[ObjectRecord]]` to be objRec
- Set `globalRec.[[GlobalThisValue]]` to be `thisValue`
- Set `globalRec.[[DeclarativeRecord]]` to be dclRec
- Set `globalRec.[[VarNames]]` to be a new, empty `List` (used to keep track of all the variables explicitly created)
- Set globalRec to be the `Environment Record` of env
- Set the environment reference of env to be **null**
- Return env

### NewModuleEnvironment(*E*)

This abstract operation creates a new module `Lexical Environment`.

The `NewModuleEnvironment` abstract operation:
- Create a new `Lexical Environment` env
- Create a new, empty, module `Environment Record` envRec
- Set envRec to be the `Environment Record` of env
- Set the environment reference of env to be `E`
- Return env