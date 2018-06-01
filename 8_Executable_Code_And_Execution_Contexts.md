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

## Realm

A `Realm` must be created before any ES code is executed. It creates, populates and provides the global object, the global `Lexical Environment`, all the built-in native objects we have available in our program, the global `this` value and also all the host environment objects.

A `Realm` is basically the "initial state" an ES program must have in order to be executed.

## Execution Contexts

An `Execution Context` is used to keep track of the runtime evaluation of some ES code.

At any point in time there can be only one `Execution Context` that is executing code, which is known as the *running execution context*.

The `Execution Context` *stack* is used to track `Execution Context`s. The running `Execution Context` is always at the top of the stack. Once control is transferred to executable code outside of the running `Execution Context`, then a new `Execution Context` is created and placed on top of the stack, thus becoming the new running `Execution Context`.

An `Execution Context` contains all the state needed to keep track of the execution of its associated code. At the very least this state is made up of 4 components:
- Code evaluation state: any state needed to perform, suspend and resume evaluation of code.
- Function: if the `Execution Context` is associated with a function (i.e. is running the code of a function body) then the value of this component is that function (called the *active function object*) Else, if the `Execution Context` is evaluating code of a script or module, the value of this component is **null**.
- Realm: the `Realm` `Record` for the current program (called the *current realm record*).
- ScriptOrModule: the script or module `Record` from which the executable code originates. This value might be **null** if the code doesn't come from neither of them (i.e. in case of an host-defined `Realm`).

The evaluation of code by the running `Execution Context` can be suspended at various points. If the `Execution Context` is suspended a new `Execution Context` can become the running `Execution Context` and start evaluating its code.

A previously suspended `Execution Context` can, at a later time, restart evaluating code from the point at which it had paused.

In a stack-like last-in first-out fashion, once the running `Execution Context` terminates, the `Execution Context` "right below it" in the stack, becomes the running `Execution Context`.
However, that's not always the case.

Usually, `Execution Context`s of ES code have 2 more components:
- LexicalEnvironment: the `Lexical Environment` of the code for this `Execution Context`.
- VariableEnvironment: the `Lexical Environment` with an `Environment Record` that holds bindings created by variable statements within this `Execution Context`.

These last 2 components are both `Lexical Environment`s. The difference is that the VariableEnvironment keeps track of all the variables declared by the `var` and `function` keywords. They are created when the containing `Lexical Environment` is instantiated (`var`s are initialized to **undefined**, `function`s correctly point to a function object). Instead the LexicalEnvironment holds both these bindings and the bindings generated by `let`, `const` and `class`, which are added to the `Lexical Environment` at its instantiation, but they can only be accessed from when the actual declaration is executed.
Then, when a block of code `{}` is met, it won't create a new `Execution Context`, but just a new `Lexical Environment`, which has an environment reference to the LexicalEnvironment of the running `Execution Context` and then actually replaces it as new LexicalEnvironment for the running `Execution Context` for as long as the block is executed.
The LexicalEnvironment is what is actually used for identifier names look-ups.

If an `Execution Context` is associated with a generator function, then it also has an additional component:
- Generator: the Generator object that the `Execution Context` is evaluating.

Most of the time only the running `Execution Context` is directly manipulated by algorithms.

An `Execution Context` is only used by the ES specification to explain internal mechanisms of the language, but it doesn't need to actually correspond to a concrete entity. That is, ES code can't interact with an `Execution Context`.

### GetActiveScriptOrModule()

This abstract operation is used to get the currently active script or module based on the running `Execution Context`. It either returns a ScriptOrModule component or **null**.

- If the `Execution Context` stack is empty, then return **null**
- From the stack, get the topmost `Execution Context` with a ScriptOrModule component which is not **null** and return this component
- If there isn't any, return **null**

### ResolveBinding(*name[, env]*)

This abstract operation takes in a `string` value `name` and optionally a `LexicalEnvironment` called `env`. Starting from `env`, it traverses the scope chain looking for a binding with an identifier name of `name`. It always returns a `Reference` value, which has a referenced name component of `name`.

The `ResolveBinding` abstract operation:
- If `env` is not present then set it to be the LexicalEnvironment component of the running `Execution Context`
- If the code being executed is running in "strict mode" then set strict to be **true** else set it to be **false**
- Return the result of calling `GetIdentifierReference` passing in `env`, `name` and strict

### GetThisEnvironment()

This abstract operation retrieves the `Environment Record` that currently supplies the value of `this`.

The `GetThisEnvironment` abstract operation:
- Get the LexicalEnvironment component of the running `Execution Context`
- Get the `Environment Record` from lex
- Call `HasThisBinding` abstract operation on envRec
- If the result is **true**, then return envRec
- Else get the environment reference of lex
- Set lex to be this value, and repeat from step 2.

The loop from step 2 will always terminate because at the end of the scope chain there is the global `Lexical Environment` which has a `this` binding.

### ResolveThisBinding()

This abstract operation retrieves the value of the `this` binding for the `Environment Record` of the `Lexical Environment` of the code associated with the running `Execution Context`.

The `ResolveThisBinding` abstract operation:
- Get the `Environment Record` that currently supplies the `this` binding by calling the `GetThisEnvironment` abstract operation
- Return the result of calling `envRec.GetThisBinding()`

`GetThisBinding` can be called on a function, or module, or global `Environment Record`.
In case of function `Environment Record`, this abstract operation looks at `envRec.[[ThisBindingStatus]]`. If the value is `"uninitialized"`, then it throws a `ReferenceError`. Else it returns the value of `envRec.[[ThisValue]]`.
For a global `Environment Record`, it simply returns the value of `envRec.[[GlobalThisValue]]`.
For a module `Environment Record`, it returns **undefined**.

### GetNewTarget()

This abstract operation retrieves the value of the `newTarget` value for the `Environment Record` of the `Lexical Environment` of the code associated with the running `Execution Context`.

The `GetNewTarget` abstract operation:
- Get the `Environment Record` that currently supplies the `this` binding by calling the `GetThisEnvironment` abstract operation
- Return the value of `envRec.[[NewTarget]]`

### GetGlobalObject()

This abstract operation returns the global object for the running `Execution Context`.

The `GetGlobalObject` abstract operation:
- Let ctx be the running `Execution Context`
- Get the current `Realm` from ctx
- Return the value of `currentRealm.[[GlobalObject]]`

## Jobs and Job Queues

A `Job` is an abstract operation that initiates an ES computation when no other computation is in progress.

A `Job` might take in an arbitrary set of parameters.

The execution of a `Job` can happen only when there is no running `Execution Context` and the `Execution Context` stack is empty.

A `PendingJob` is a request for the future execution of a `Job`. It is an internal `Record` that holds all the information necessary for running the associated `Job`.

`Job`s always execute to completion. That is no `Job` can be initiated while another `Job` is running.

`Job`s can cause other `Job`s to be enqueued in the `Job` queue. That is a `Job` can generate `PendingJob Record`s.

A `PendingJob` has 5 fields:
- `[[Job]]`: the name of a `Job`. It's the abstract operation that will be executed when this `PendingJob` is taken from the queue.
- `[[Arguments]]`: the `List` of arguments passed to the `[[Job]]` abstract operation.
- `[[Realm]]`: the `Realm Record` for the initial `Execution Context` when this `PendingJob` is initiated.
- `[[ScriptOrModule]]`: the script or module for the initial `Execution Context` when this `PendingJob` is initiated.
- `[[HostDefined]]`: field reserved for any additional information the host environment needs to give the `PendingJob`. The default value is **undefined**.

A `Job Queue` is a queue (first-in first-out) of `PendingJob`s.

There can be many `Job Queue`s and each has its own name. Every ES implementation must at least provide 2 `Job Queue`s:
- `ScriptJobs`: contains the `Job`s that validate and evaluate ES script and module source text.
- `PromiseJobs`: contains the `Job`s that are responsible for settling Promises.

A request for the future execution of a `Job` is made by enqueueing a `PendingJob` for that `Job` in a `Job Queue`.
When there is no running `Execution Context` and the `Execution Context` stack is empty, then the first `PendingJob` is removed from a `Job Queue`. The information it contains is used to generate a new `Execution Context` to start the execution of the `[[Job]]` abstract operation. This `Execution Context` is pushed into the stack, and becomes the running `Execution Context`.

Implementation of the ES language is free to choose the order in which `Job Queue`s are served, and also what happens when there are no running `Execution Context`s and all the `Job Queue`s are empty.

### EnqueueJob(*queueName, job, arguments*)

This abstract operation takes in a `Job Queue` name as a `string` value `queueName`, a specific `Job` as `job`, and a `List` of arguments required by `job`.
This abstract operation generates a new `PendingJob` for `job` and pushes in the `Job Queue` named `queueName`.

The `EnqueueJob` abstract operation:
- First retrieve from the running `Execution Context` its `Realm` and `ScriptOrModule` components
- Create a new `PendingJob{ [[Job]]: job, [[Arguments]]: arguments, [[Realm]]: callerRealm, [[ScriptOrModule]]: callerScriptOrModule, [[HostDefined]]: undefined }`
- Perform any host environment process on this `PendingJob`
- Add `PendingJob` at the back of the `Job Queue` named `queueName`
- Return `NormalCompletion(empty)`

## Agents and Agent Cluster

An `agent` is a set of `Execution Context`s, an `Execution Context` stack, a running `Execution Context`, a set of named `Job Queue`s, and `Agent Record` and an executing thread.

Apart from the executing thread, whatever belongs to an `agent` is personal to that `agent`. Several `agent`s can share the same executing thread.

The executing thread of an `agent` executes the `Job`s in the `Job Queue`s of that `agent` by using its `Execution Context` stack independently of any other `agent` that shares that thread. The exception to this is that one of the `agent`s sharing the executing thread has a `[[CanBlock]]` property set to **true**.

When the executing thread executes the `Job`s in the `Job Queue` of a certain `agent`, that `agent` is called the *surrounding `agent`*, that is, it is the `agent` that provides the code with the running `Execution Context`, the `Execution Context` stack, the named `Job Queue`s and the `Agent Record` fields.

Just like many other specification concepts, an `agent` is an abstract entity used to explain the internal mechanisms of the ES language and doesn't need to be an actual entity in any ES implementation.

An `agent cluster` is a set of `agent`s that can communicate by operating on shared memory.

Some `agent`s can communicate by message passing, but if they don't share the same memory then they aren't is the same `agent cluster`.

Every `agent` belongs to only 1 `agent cluster`.

Each `agent` must have an `Agent Record` with a `[[Signifier]]` property that uniquely identifies that `agent` in its `agent cluster`.
