# Ordinary objects internal methods and slots

#### [[Prototype]]

All ordinary objects have an internal slot called `[[Prototype]]` which is either **null** or another object which is used for property resolution delegation.

All the properties on the `[[Prototype]]` object are inherited by the target object, so that we can access them from the target object.

**Note:** delegation is used by both `[[Get]]` and `[[Set]]` operations, but we can't actually set a property on the `[[Prototype]]` by using the target object.
What happens is that if we try to create a property on a target object, first its `[[Prototype]]` is consulted and if a property with the same name exists somewhere in there then 1 of 3 things might happen:
- if it is a data property with `writable: true`, then a new property with the same name is created on the target object
- if it is a data property with `writable: false`, then the operation fails
- if it is an accessor property, then the setter is called, but it will operate starting from the target object

#### [[Extensible]]

All ordinary objects have an `[[Extensible]]` internal slot, which is a `boolean` value. This slot controls whether or not new properties can be added to the target object.
Setting `[[Extensible]]` to **false** is a one-way action that cannot be reversed.

**Note:** when `[[Extensible]]` is **false**, we can't reassign the `[[Prototype]]` of the target object.

## Internal methods

Each internal method is invoked directly on the target object and it delegates to an abstract operation passing in the object itself. This is so if the same internal method is called on an exotic object, that object implementation of the method can be used.

Usually the abstract operation for a method `[[MethodName]]` is `OrdinaryMethodName`.

### [[GetPrototypeOf]]()

This method delegates to `OrdinaryGetPrototypeOf`, which simply retrieves the value of the object `[[Prototype]]` slot.

### [[SetPrototypeOf]](*V*)

This method delegates to `OrdinarySetPrototypeOf` and is used to change the value of the `[[Prototype]]` of the target object.

`OrdinarySetPrototypeOf` abstract operation:
- First retrieve the value of `O.[[Extensible]]`
- Then retrieve the value of the current `[[Prototype]]` object, `O.[[Prototype]]`
- If `V` and this value are the same, then return **true**
- Else, if `O` is not extensible, return **false**
- Else, create an alias p for `V` and a boolean done set to **false**
- While done is **false**
  - if p is **null** set done to **true**
  - else if p and `O` are the same value, return **false**
  - else
    - if p doesn't have the default implementation of `[[GetPrototypeOf]]` set done to **true**
    - else set p to `p.[[Prototype]]`
- Finally, set `O.[[Prototype]]` to `V` and return **true**

**NB:** the loop is used to avoid circular references in a `[[Prototype]]` chain
formed of objects with the default implementation of `[[GetPrototypeOf]]`.

### [[IsExtensible]]()

This method delegates to `OrdinaryIsExtensible`, which just retrieves the value of the `[[Extensible]]` slot.

### [[PreventExtensions]]()

This method delegates to `OrdinaryPreventExtensions` which simply sets `[[Extensible]]` to **false**.

### [[GetOwnProperty]](*P*)

This method is called with a property key `P` and it delegates to `OrdinaryGetOwnProperty` which retrieves the `Property Descriptor` for `P`.

`OrdinaryGetOwnProperty` abstract operation:
- If `O` doesn't have an own property with the name of `P`, then return **undefined**
- Else create an empty `Property Descriptor` D
- If `P` is a data property then
  - set `D.[[Value]]` to `P`'s `[[Value]]`
  - set `D.[[Writable]]` to `P`'s `[[Writable]]`
- Else `P` must be an accessor property then
  - set `D.[[Get]]` to `P`'s `[[Get]]`
  - set `D.[[Set]]` to `P`'s `[[Set]]`
- Set `D.[[Enumerable]]` to `P`'s `[[Enumerable]]`
- Set `D.[[Configurable]]` to `P`'s `[[Configurable]]`
- Return `D`

### [[DefineOwnProperty]](*P, Desc*)

This method is called with a valid property key `P` and a `Property Descriptor` `Desc` and it tries create `P` on the target object. The process is delegated to the abstract operation `OrdinaryDefineOwnProperty`.

`OrdinaryDefineOwnProperty` retrieves the own property `P` `Property Descriptor` and the value of `O.[[Extensible]]`, then calls `ValidateAndApplyPropertyDescriptor` passing in `O`, `P`, `extensible`, `Desc` (the `Property Descriptor` passed to `[[DefineOwnProperty]]`) and `current` (the `Property Descriptor` of `P`).

`ValidateAndApplyPropertyDescriptor` abstract operation checks if the property `P` does not exists on `O`. If that's the case and `O` is not extensible, then the operation returns **false**. Else it creates a new own property `P` on `O` with a `Property Descriptor` which takes its values from `Desc`.
Else, it validates `Desc` based on the values of `current` fields and if appropriate, updates `P` `Property Descriptor` with the values of `Desc`.

### [[HasProperty]](*P*)

This method is called with a property key `P` and it is a recursive method (in fact it traverses the `[[Prototype]]` chain) that delegates to `OrdinaryHasProperty`.

`OrdinaryHasProperty` abstract operation:
- Get the `Property Descriptor` of `P` calling `O.[[GetOwnProperty]](P)`
- If this value is not **undefined**, then return **true**
- Else get `O`'s `[[Prototype]]` object
- If this object is not **null** then call `parent.[[HasProperty]](P)`
- Else return **false**

### [[Get]](*P, Receiver*)

This method is called with a valid property key `P` and any ES language value `Receiver` and it calls the abstract operation `OrdinaryGet`. It is recursive method because it traverses the `[[Prototype]]` chain of `O`.

The `OrdinaryGet` abstract operation first gets the `Property Descriptor` of `P`. If that's **undefined**, that is `P` doesn't exists on `O`, then it delegates the look-up to the `[[Prototype]]` linked object. If there isn't any, it returns **undefined**.
If finally `P` is found and it is a data property, then the `[[Value]]` of its `Property Descriptor` is returned. Else, the getter method is called passing in the `Receiver` as context.

### [[Set]](*P, V, Receiver*)

This method tries to set the value `V` on the property `P` of `O`. It is a recursive method because if `P` does not exists on `O` then its `[[Prototype]]` chain is traversed. If no property is found with the name of `P`, then a new property `P` is created on `O`. Else, if the property `P` exists at some level in the chain 1 of 3 things can happen:
- if `P` is a data property with `[[Writable]]` to **true** then a new property `P` is created on `O`
- if `P` is a data property with `[[Writable]]` to **false** then the operation fails and `[[Set]]` returns **false**
- if `P` is an accessor property then its setter is called but the set operation starts from the object `O`, rather than from the object owning `P`

`[[Set]]` delegates to `OrdinarySet`. This abstract operation only retrieves the `Property Descriptor` of `P` (which might be **undefined**) and calls `OrdinarySetWithPropertyDescriptor` which is the operation that actually performs the set.

### [[Delete]](*P*)

This method is called with a property key `P` and delegates to `OrdinaryDelete` which tries to delete property `P` from `O`.

`OrdinaryDelete` abstract operation:
- Get the `Property Descriptor` of `P`
- If it is **undefined** then return **true**
- Else if `desc.[[Configurable]]` is **true** remove the own property `P` from `O` and return **true**
- Else return **false**

### [[OwnPropertyKeys]]()

This method returns a list of all the keys of the own properties of `O`. It calls the `OrdinaryOwnPropertyKeys` abstract operation.

`OrdinaryOwnPropertyKeys` first creates an empty `List`, then retrieves all the properties on `O` which are indices and adds them to the `List` in ascending numeric index order. Then retrieves all the own properties with a property key which is a string and adds them to the `List` in ascending chronological order of creation. Finally it does the same things with symbol properties and returns the `List`.

### ObjectCreate(*proto[, internalSlotsList]*)

This abstract operation is called with argument `proto` being either an object or **null** and is used to create a new ordinary object `[[Prototype]]` linked to `proto`. Optionally an `internalSlotsList` might be passed, which is a `List` of all the names of the internal slots with which the object should be initialized.

`ObjectCreate` abstract operation:
- If there is no `internalSlotsList` set it to a new empty `List`
- Create a new object and add to it all the internal slots in `internalSlotsList`
- Set the essential internal methods of this object to have the default implementation
- Set `object.[[Prototype]]` to be `proto`
- Return the object

# Function objects

Functions encapsulate code closed over a `Lexical Environment` and support the dynamic evaluation of that code.
The code within a function can either be strict mode or non-strict mode.

A function is an ordinary object, thus it has the same ordinary internal slots and methods.
Other than those, it also has additional ones.

### [[Environment]]

It is the `Lexical Environment` that the function was closed over, that is where the function was declared.

When the function is called and its `Lexical Environment` is created, its outer environment reference is set to be the function's `[[Environment]]`.

### [[FormalParameters]]

It is the root `Parse Node` of the source text that defines the function formal parameters list.

### [[FunctionKind]]

It is a string that specifies if the function is a plain function (`"normal"`), a class (`"classConstructor"`), a generator function (`"generator"`) or an async function (`"async"`).

### [[ECMAScriptCode]]

It is the root `Parse Node` of the source text that defines the function body.

### [[ConstructorKind]]

It's a `string` value, either `"base"` or `"derived"` (it's a child class).

### [[Realm]]

It is the `Realm Record` in which the function was created. It provides the native objects the function can access.

### [[ScriptOrModule]]

It's the `Record` representing the script or module in which the function was created.

### [[ThisMode]]

Defines how the `this` keyword is evaluated for calls of the function. It can either be `lexical` or `strict` or `global`.

`lexical` means that the value of `this` will be the value of `this` of the lexically enclosing function (that is where our function was declared) (this is the case for arrow functions).

`strict` means that `this` will be any value the function specifies.

`global` means that if the function gives an **undefined** value for `this`, the global object is used instead.

### [[Strict]]

It's **true** if the function has to run in strict mode.

### [[HomeObject]]

If the function uses the `super` keyword, then `[[HomeObject]]` provides the object whose `[[Prototype]]` is the object in which `super` look-ups begin.

### [[Call]](*thisArgument, argumentsList*)

All functions have the internal `[[Call]]` method. This method executes the body of the function `F`.

`[[Call]]` internal method:
- If `F.[[FunctionKind]]` is `"classConstructor"`, then throw a `TypeError`
- Get the currently running `Execution Context` as callerContext
- Invoke `PrepareForOrdinaryCall` abstract operation passing in `F` and **undefined**. This abstract operation creates a new `Execution Context`, sets it as the running `Execution Context` and returns it, which is then saved as calleeContext
- Call `OrdinaryCallBindThis` passing in `F`, calleeContext and `thisArgument`. This abstract operation sets the value of `this` for the current call of `F`
- Call `OrdinaryCallEvaluateBody` passing in `F` and `argumentsList`, which executes the body of the function and returns a `Completion Record` as result
- Remove calleeContext from the `Execution Context` stack and sets again callerContext as the running `Execution Context`
- If `result.[[Type]]` is `return`, then return `NormalCompletion(result.[[Value]])`
- Else, if result is an abrupt completion, propagate it
- Else, return `NormalCompletion(undefined)`

### PrepareForOrdinaryCall(*F, newTarget*)

This abstract operation is called with a function `F` and a `newTarget` which is either **undefined** or an object.

It creates a new `Execution Context` for `F`, suspends the running `Execution Context` and sets `F`'s as the new running `Execution Context`. Finally return it.

### OrdinaryCallBindThis(*F, calleeContext, thisArgument*)

This abstract operation sets the `[[ThisValue]]` field of the `Environment Record` of `F` `Lexical Environment` based on the `[[ThisMode]]` internal slot of `F`.

For `strict`, it simply sets it to be `thisArgument`.
For `global`, if `thisArgument` is **undefined** or **null**, the it is set to be the `globalObject`.
For `lexical`, the `[[ThisValue]]` field is not set, but rather the operation immediately returns `NormalCompletion(undefined)`.

### OrdinaryCallEvaluateBody(*F, argumentsList*)

This abstract operation returns the result of the execution of `EvaluateBody` on the parsed code given by `F.[[ECMAScriptCode]]`, passing in `F` and `argumentsList`.

### [[Construct]](*argumentsList, newTarget*)

This internal method is called with a `List` `argumentsList` which contains ES language values, but might also be empty, and an object `newTarget`.

`[[Construct]]` internal method:
- First create a new object `[[Prototype]]` linked to `newTarget` **prototype**
- Invoke `PrepareForOrdinaryCall` passing in `F` and `newTarget`, which creates a new `Execution Context` for `F` and sets it as the running one
- Call `OrdinaryCallBindThis` which sets the value of `this` to be the newly created object
- Call `OrdinaryCallEvaluateBody` which executes the function body and returns a `Completion Record`
- If the result is of `[[Type]]` `return` then
  - if `[[Value]]` is an object, then return it
  - else if `F.[[ConstructorKind]]` is `"base"` return `this` (the object created in step 1)
  - else `F.[[ConstructorKind]]` is `"derived"` and if `[[Value]]` is not **undefined**, throw a `TypeError`
- Else, it the result is an error, propagate it
- Else return `envRec.GetThisBinding()`

If `[[ConstructorKind]]` is `"base"`, if the `constructor` function of the class returns a non-object value, that return is ignored and instead the newly created object (stored in the `this` keyword) is returned. Else, if the `constructor` returns an object, then that object is returned.

But if `[[ConstructorKind]]` is `"derived"`, that is if this class is a child class, then if the `constructor` function returns a value which is not **undefined** nor an object, then a `TypeError` is thrown.

### FunctionAllocate(*functionPrototype, strict, functionKind*)

This abstract operations creates a function object adding to it all the internal slots a method of the ordinary functions, then returns it.
`functionPrototype` is the object to be used as `[[Prototype]]`, while `strict` is whether or not the function should run in strict mode.
Finally, if `functionKind` is normal, then this function will also have a `[[Construct]]` method.

### FunctionInitialize(*F, kind, ParameterList, Body, Scope*)

This abstract operation takes in a function object `F` and initializes its internal slots.

`FunctionInitialize` abstract operation:
- First set the `length` of `F` to be the number of parameters given by `ParameterList`
- Set `F.[[Environment]]` to be `Scope`
- If `kind` is `Arrow` set `F.[[ThisMode]]` to be `lexical`
- Else if `F.[[Strict]]` is **true** set `F.[[ThisMode]]` to be `strict`
- Else set `F.[[ThisMode]]` to be `global`
- Return `F`

### FunctionCreate(*kind, ParameterList, Body, Scope, Strict[, prototype]*)

This abstract operation simply calls `FunctionAllocate` and `FunctionInitialize` passing in the right arguments.

### MakeConstructor(*F[, writablePrototype[, prototype]]*)

This abstract operation creates a new ordinary object with an own `constructor` property which points at `F`. Then it creates an own `prototype` property on `F` which points at this object.

### SetFunctionName(*F, name[, prefix]*)

This abstract operation creates a `name` property on `F`. If the argument `name` is a string, then it is used as value for the `name` property. If instead `name` is a symbol, then `name.[[Description]]` is used as value for `name`.

### SetFunctionLength(*F, length*)

This abstract operation creates a `length` property on `F`.

### FunctionDeclarationInstantiation(*func, argumentsList*)

This abstract operation creates all the bindings for the `Environment Record` of `func`.

When a function is invoked a new `Execution Context` for that function is created which contains an `Environment Record` to keep track of all the bindings for the identifiers declared in the body of the function.
`FunctionDeclarationInstantiation` initializes both the formal parameters of the function `func`, all its internal function declarations and `var` variable declarations. That is, they are added on the `Environment Record` and a value is assigned to them.
All other bindings are created here but initialized during the evaluation of the function body.
If `func` parameters include default parameters assignments, then 2 `Environment Record`s are created, one holding the bindings of the parameters and one holding all the other bindings.
This separation of `Environment Record`s is needed so that if we use default parameters and assign a function, that function won't be able to see the variables declared in `func` body.
