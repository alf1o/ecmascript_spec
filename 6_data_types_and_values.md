# Data types and values

## convention

An algorithm is represented as a numeric sequence of steps.
Some algorithms, like *abstract operations*, might use a function-like syntax (i.e. `name(arg1, arg2)`).

Algorithms manipulate *values*.
Each value has an associated *type*.

Types are used to separate values from other values, based on their intrinsic set of characteristics (the ways that a value can be used).
For example, both the programmer and the Engine would treat the value `42` differently than the value `"42"`. Thus these 2 values have different type.

There are both ES language types and specification types.

`Type(x)` is used as shorthand for "the type of x".

## ES language types

*An ES language type corresponds to values that are directly manipulated by the programmer by using the ES language.*

An ES value, is a value with an associated ES language type.

The ES language types are `undefined`, `null`, `boolean`, `string`, `number`, `symbol`, `object`.

### `undefined`

The `undefined` type has only one value, called **undefined**.

Any variable that has not been assigned a value has the value **undefined**.

### `null`

The `null` type has only one value, called **null**.

**null** represents the intentional absence of any object value.

### `boolean`

The `boolean` type represents a logical entity.
It can be either **true** or **false**.

### `string`

The `string` type is the set of all ordered sequences of zero or more 16-bit unsigned integer values up to a maximum of `2^53 - 1` elements.

Hence:
- A string is an ordered sequence.
- Each element in the string occupies a specific position in the string, identified by a non-negative integer (index). The first element has index `0`, the second element has index `1`, and so on.
- An element is a 16-bit unsigned integer value.
- The `length` of a string is the number of elements in the string.
- A string can be empty, thus containing no elements.
- There can be at max `2^53 - 1` elements in a single string.

The `string` type is used to represent textual data. So, each element in the string is treated as an UTF-16 code unit value.

### `symbol`

The `symbol` type is the set of all non-`string` values that can be used as the key of an object property.

Each possible `symbol` value is *unique* and *immutable*.

Each `symbol` value has an associated value called `[[Description]]` that is either **undefined** or a `string` value.

The ES language provides some built-in `symbol` values. Such symbols are referred to in the form `@@name`.

Some of these "well-known" symbols are:
- `@@hasInstance`: method that determines whether a constructor recognizes an certain object as being its instance. It's used by the `instanceof` operator.
- `@@iterator`: method that returns the default iterator for an object. It's used by the `for..of` statements.
- `@@species` : determines the constructor function to be used to create the derived objects. It's used by children classes to specify whether their instances should be instead instances of their parent class.

**NB:** to access a property located at a `symbol` key we use *computed property name* syntax (`[Symbol.symbolName]`).

### `number`

The `number` type represents values in the double precision 64-bit format IEEE 754-2008.
The maximum value that can be represented is `1.798e+308` (`Number.MAX_VALUE`), while the minimum value is `5e-324` (`Number.MIN_VALUE`).
The range of values that can be unambiguously represented goes from `-(2^53 - 1)` (`Number.MAX_SAFE_INTEGER`) up to `2^53 - 1` (`Number.MIN_SAFE_INTEGER`).

The `number` type contains some special values.

All the "Not-a-Number" values of the double precision format are represented as a single special `NaN` value in the ES langauge.

To the ES language all `NaN` values are indistinguishable, but some implementations of the language might actually make distinctions.

The special values `Infinity` and `-Infinity` are used to represent any value that goes beyond the maximum/minimum representable value.

The ES language has both a `+0` and a `-0`.

**NB:** some operators only work with numbers in the range `-2^31`-`2^31 - 1` or `0`-`2^16 - 1`. These operators accept any `number` value, but first operate the conversion to a `number` value in the specified range.

### `object`

An object is a collection of zero or more properties each with attributes that determine how a property can be used.

Each property is either a:
- `data property` : associates a key value with an ES language value and has a set of `boolean` attributes.
- `accessor property` : associates a key value with 1 or 2 accessor functions and a set of `boolean` attributes. The accessor functions (`get`, `set`) are used to retrieve an ES language value.

Properties are accessed by using key values. A key value is either a `string` or a `symbol` value.
All `string` and `symbol` values (including the empty string) are valid key values.

An *integer index* is a `string` value that is a canonical numeric `string` value and whose numeric value is a non-negative integer.
An *array index* is an integer index in the range `0`-`2^32 - 1`.

**NB:** a canonical numeric string is a string that can be converted to a number.

Properties can be accessed either by a `[[Get]]` algorithm, which is used for value retrieval, or by a `[[Set]]` algorithm, which is used for assignment.
`[[Get]]` and `[[Set]]` will access both *own properties* (directly on the object) and *inherited property* (on the `[[Prototype]]` chain of the object).
Each key in an object must be unique.

Each object is a collection of properties. But the ES language defines 2 kinds of objects that differ in their semantics for accessing and manipulating their properties:
- *ordinary objects* have the default behavior for the essential internal methods that must be supported by all objects
- *exotic objects* do not have the default behavior for one or more of the essential internal methods

#### property attributes

Attributes specify how each property can be used.

The attributes for a data property are:
```
  Name           |  Domain          |  Description                             |
--------------------------------------------------------------------------------
[[Value]]        | any ES language  | the value retrieved by a                 |
                 | type             | [[Get]] access of the property           |
--------------------------------------------------------------------------------
[[Writable]]     | boolean          | if false, attempts by the code to change |
                 |                  | the property [[Value]] attribute by using|
                 |                  | [[Set]] will fail                        |
--------------------------------------------------------------------------------
[[Enumerable]]   | boolean          | if true, the property will be enumerated |
                 |                  | by a for-in enumeration. Else it won't   |
--------------------------------------------------------------------------------
[[Configurable]] | boolean          | if false, attempts to delete the property|
                 |                  | or change its attributes (other than     |
                 |                  | [[Value]], or set [[Writable]] to false) |
                 |                  | will fail.                               |
```

The attributes for an accessor property are:
```
  Name           |  Domain          |  Description                             |
--------------------------------------------------------------------------------
[[Get]]          | undefined or     | if the value is an object, it must be a  |
                 | object           | function object. The function is called  |
                 |                  | with no arguments to retrieve the        |
                 |                  | property value each time a [[Get]] access|
                 |                  | of the property is performed.            |
--------------------------------------------------------------------------------
[[Set]]          | undefined or     | if the value is an object it must be a   |
                 | object           | function object. It is called with only  |
                 |                  | one argument, which is the value to be   |
                 |                  | assigned to the property, each time a    |
                 |                  | [[Set]] access of the property is        |
                 |                  | performed. The operation might affect the|
                 |                  | value returned by subsequent calls of the|
                 |                  | property [[Get]] method.                 |
--------------------------------------------------------------------------------
[[Enumerable]]   | boolean          | if true, the property will be enumerated |
                 |                  | by a for-in enumeration. Else it won't   |
--------------------------------------------------------------------------------
[[Configurable]] | boolean          | if false, attempts to delete the property|
                 |                  | or change its attributes (other than     |
                 |                  | [[Value]], or set [[Writable]] to false) |
                 |                  | will fail.                               |
```

By default a property is created as a data descriptor.

#### internal methods and internal slots

The semantics of objects (the way the work) are specified by algorithms called *internal methods*.
Internal methods specify the runtime behavior of an object.
Each object has its own internal methods and must behave as they specify.
(The way in which this is accomplished is implementation-dependent).

Internal methods are *polymorphic*, which means that different object values can perform different algorithms when the same internal method is called on them.

The actual object upon which the internal method is called takes the name of *target* object. If an algorithm tries to use an internal method on an object that doesn't support it, a `TypeError` is thrown.

*Internal slots* correspond to an internal state associated with each object and used by various algorithms. Such state can consists of ES language values or specification values.
Internal slots are not object properties and are not inherited.
By default internal slots are created during the process of object creation and cannot be dynamically added and their initial value is **undefined**.

Internal methods and internal slots are identified by names wrapped in `[[]]`.

There is a set of *essential internal methods* that are applicable to all objects. Every object must have algorithms for these methods, but not all objects use the same algorithm for a specific method.
An object which implements the default behavior for these methods is an ordinary object.

Each internal method might accept a list of parameters, but each method has always access to the target object.
An internal method might have an explicit return value, but every method implicitly returns a `Completion Record` (specification type).

The essential internal methods are:
* `[[GetPrototypeOf]]` : `() -> object|null` : determine the object that provides inherited properties for the target. **null** means that there is none.
* `[[SetPrototypeOf]]` : `(object|null) -> boolean` : links the target with an object that will provide inherited properties. Passing **null** means that there won't be any inherited property. Returns **true** if the operation was successful, else returns **false**.
* `[[IsExtensible]]` : `() -> boolean` : determines if it's permitted to add additional properties to the target.
* `[[PreventExtensions]]` : `() -> boolean` : control if new properties can be added to the target. Returns **true** if the operation was successful, else **false**.
* `[[GetOwnProperty]]` : `(propertyKey) -> undefined|PropertyDescriptor` : if the target has an own property whose key is `propertyKey` than returns the `Property Descriptor` (specification type) for that property. Else returns **undefined**.
* `[[DefineOwnProperty]]` : `(propertyKey, PropertyDescriptor) -> boolean` : create (or alter if it already exists) the own property with the key of `propertyKey` and set its `PropertyDescriptor` to be the specified one. If the operation was successful returns **true**, else returns **false**.
* `[[HasProperty]]` : `(propertyKey) -> boolean` : if the target has an own property or an inherited property with the key `propertyKey` returns **true** else returns **false**.
* `[[Get]]` : `(propertyKey, Receiver) -> any` : returns the value of the property (own or inherited) with the key `propertyKey`. If any code must be executed to retrieve the property value, that code will receive `Receiver` as `this` value.
* `[[Set]]` : `(propertyKey, value, Receiver) -> boolean` : sets the value of the key `propertyKey` to be `value`. If any code must be executed, that code will receive `Receiver` as `this` value. Returns **true** if the property value was set, **false** if it could not be set.
* `[[Delete]]` : `(propertyKey) -> boolean` : removes from the target the own property with the key `propertyKey`. If the property could not be deleted and is still present returns **false**. If the deletion was successful or the property was not present returns **true**.
* `[[OwnPropertyKeys]]` : `() -> List of property keys` : returns a `List` (specification type) with elements being the keys of the own properties of the target.

Functions are *callable objects*. They have 2 more internal essential methods:
* `[[Call]]` : `(any, List of any) -> any` : executes the code associated with the target. The arguments to this internal method are the value of `this` for the specific invocation and a list of the arguments actually passed to the function.
* `[[Construct]]` : `(List of any, object) -> object` : creates an object. This internal method is invoked when a function is called via a `new` expression. Such function is then called a *constructor*. The first argument is the list of arguments actually passed to the function. The second argument is the object to which the `new` operator was initially applied.

**NB:** some functions might not have the internal `[[Construct]]` method, so they can't be constructor functions.

Each internal method, when implemented, must conform to a list of invariants.
**invariant is a condition that can be relied upon to be always true**.

## primitive types

All the built-in types but `object` are called *primitive types*. A value that has a type which is a primitive type is a *primitive value*.

An object is a location in memory where data can be stored.
In ES an object refers to a *data-structure* containing both the data and the behavior associated with that data (methods).

A primitive value is data that is not an object, so it doesn't provide any methods.

A primitive value is **immutable** and it is data that is represented directly at the lowest level of the language implementation.

## ES specification types

*A specification type is a meta-value that is used by algorithms to describe the semantics (way in which something can be used) of ES language constructs and types.*

Specification values do not necessarily correspond to any specific entity of the ES implementation, meaning that some of them are just abstractions and not concrete values with which we can interact.

Specification values cannot be stored as properties of objects nor as values inside variables.

The specification types are : `List`, `Record`, `Set`, `Relation`, `Reference`, `Completion`, `Property Descriptor`, `Lexical Environment`, `Environment Record`, `Data Block`.

### `List` and `Record`

The `List` type is used to explain the evaluation of argument lists in `new` expressions, function calls and anywhere an ordered list of values is needed.

The `List` type is used to describe an ordered list of elements.

A `List` value is an ordered sequence of elements. Each element is an individual value. The sequence can be of any length and elements can be accessed using indices.

`arguments[2]` is a shorthand for saying the 3rd element of the `List` called `arguments`.

`<< 1, 2 >>` represents a `List` of 2 elements, both initialized with a certain value. An empty `List` is represented as `<< >>`.

The `Record` type is used to describe data aggregation within algorithms.

A `Record` value consists of one or more named fields, the value of which is either an ES language value or an abstract value. Field names are enclosed in `[[]]`.

`{ [[Field1]]: 42, [[Field2]]: false, [[Field3]]: empty }` defines a `Record` value with 3 fields, each initialized to a specific value. Fields are not ordered.

If `R` is the name of a `Record` value, then `R.[[Field2]]` is a shorthand for saying the field of `R` named `[[Field2]]`.

### `Set` and `Relation`

The `Set` type is used to describe a collection of unordered elements for use in the *memory model*.

A `Set` value is a collection of elements, where no elements appears more than once. Elements can be added or removed from a `Set`.

`Set`s support operations like union, intersection and subtraction.

The `Relation` type is used to represent constraints on a `Set` value.

A `Relation` value is a `Set` of ordered pairs of values from the `Relation` domain. For example a `Relation` on events is a `Set` of ordered pairs of events.

If `R` is a `Relation` and `a` and `b` are 2 values in `R` value domain, then `a R b` is a shorthand for saying that the ordered pair `(a, b)` is a member of `R`.

Given certain conditions, the smallest `Relation` that satisfies them is said to be *least* with respect to those conditions.

A *strict partial order* is a `Relation` value `R` that satisfies the conditions of *irreflexivity* and *transitivity*:
1. For all `a`, `b` and `c` in `R` domain:
  a. it's **not** the case that `a R a` (irreflexivity)
  b. if `a R b` and `b R c` then `a R c` (transitivity)

A *strict total order* is a `Relation` value `R` that satisfies the conditions of *totality*, *irreflexivity* and *transitivity*:
1. For all `a`, `b` and `c` in `R` domain:
  a. `a R b` or `b R a` (totality)
  b. it's **not** the case that `a R a` (irreflexivity)
  c. if `a R b` and `b R c` then `a R c` (transitivity)

### `Completion Record`

The `Completion` type is a `Record` used to explain the runtime propagation of values and control flow (i.e. the behavior of statements like `continue`, `break`, `return`, `throw`) that perform non-local transfer of control.

A `Completion` value is a `Record` value whose fields are:
```
  Field     | Value               | Meaning                                    |
--------------------------------------------------------------------------------
[[Type]]    | one of normal, break| the type of Completion Record that occurred|
            | continue, return,   |                                            |
            | or throw            |                                            |
--------------------------------------------------------------------------------
[[Value]]   | any ES language     | the value that was produced                |
            | value or empty      |                                            |
--------------------------------------------------------------------------------
[[Target]]  | string or empty     | the target label for directed control      |
            |                     | transfer                                   |
```
Such `Record` is called a `Completion Record`.

Basically a `Completion Record` explains how something interrupted execution, with what value, and what will execute next.

An *abrupt completion* is a completion with a `[[Type]]` value which is not `normal`.

`NormalCompletion` is an *abstract operation*.
The syntax `NormalCompletion(argument)` is a shorthand for saying return `Completion{ [[Type]]: normal, [[Value]]: argument, [[Target]]: empty }`.

#### implicit `Completion` values

Algorithms often implicitly return a `Completion Record` with a `[[Type]]: normal`.
For example, the syntax `return Infinity` is a shorthand for saying return `NormalCompletion(Infinity)`.

But a `Completion Record` can also be explicitly returned.

If the return expression is a call to an abstract operation then the `Completion Record` of that abstract operation is returned.

`Completion` is an abstract operation and `Completion(completionRecord)` is used to emphasize that a previously computed `Completion Record` is now returned.

`Completion(completionRecord)` follows 2 steps:
1. assert that `completionRecord` is a `Completion Record`
2. return `completionRecord`

A return statement without a value is the same as saying `NormalCompletion(undefined)`.

#### thrown exceptions

An algorithm that throws an exception returns a `Completion Record` with `[[Type]]: throw`.

For example, throw a `TypeError` exception is a shorthand for return `Completion{ [[Type]]: throw, [[Value]]: TypeError object, [[Target]]: empty }`.

#### abrupt returns

An algorithm step which is `ReturnIfAbrupt(argument)` is equivalent to:
1. if `argument` is an *abrupt completion* return `argument`
2. else if `argument` is a `Completion Record`, let it be `argument.[[Value]]`

Basically this means, if we have an abrupt completion return it, else if it is a normal completion return its `[[Value]]`.

**Reminder:** an abrupt completion is any `Completion Record` with a `[[Type]]` which is not `normal`.

An algorithm step which is `ReturnIfAbrupt(AbstractOperation())` is the same as:
1. let `hygienicTemp` be `AbstractOperation()`
2. if `hygienicTemp` is an abrupt completion return `hygienicTemp`
3. else if `hygienicTemp` is a `Completion Record`, let it be `hygienicTemp.[[Value]]`

Basically this means, execute the `AbstractOperation()`. If it returns an abrupt completion, return it. Else return its `[[Value]]`.

An algorithm step which is `AbstractOperation(ReturnIfAbrupt(argument))` is equivalent to:
1. if `argument` is an abrupt completion return `argument`
2. if `argument` is a `CompletionRecord`, let it be `argument.[[Value]]`
3. let result be `AbstractOperation(argument)`

#### `UpdateEmpty(completionRecord, value)`

`UpdateEmpty` is an abstract operation.
When invoked in the form `UpdateEmpty(completionRecord, value)` it does:
1. assert if `completionRecord.[[Type]]` is either `return` or `throw`, then assert that `completionRecord.[[Value]]` is not `empty`
2. if that's the case return `Completion(completionRecord)`
3. return `Completion{ [[Type]]: completionRecord.[[Type]], [[Value]]: value, [[Target]]: completionRecord.[[Target]] }`

### `Reference` type

The `Reference` type is used to explain the behavior of operations like `delete`, `typeof`, assignments, `super` and more. For example, the left-hand side of an assignment operation is expected to return a `Reference`.

A `Reference` is a resolved name or property binding.

A `Reference` has 3 *components*:
- base value component, is either `undefined`, `object`, `boolean`, `string`, `number`, `symbol` or `Environment Record`. If it is `undefined` it mean that the `Reference` could not be resolved to a binding.
- referenced name component, is either a `string` or `symbol`
- strict reference flag, is a `boolean`

*The base value is the context in which the referenced name lives.* If the base value component is **undefined**, then there's not actual binding corresponding to the referenced name. If the base value is a primitive value or an object then the referenced name is a property on that object (in case of a primitive value we have boxing, that is the primitive is automatically wrapped in an object before property access). If the base value is an environment record, that means the binding of the referenced name is variable, and the environment record is the execution context in which that variable lives.

A `Super Reference` is a `Reference` that is used to represent a name binding that was expressed using `super`. Such `Reference` has an additional *thisValue component* and its *value component* will never be an `Environment Record`.

The abstract operations used to operate on `Reference`s are `GetValue`, `PutValue`, `GetThisValue` and `InitializeReferenceBinding`.

#### `Property Descriptor` type

The `Property Descriptor` type is used to describe the attributes of object properties.

A `Property Descriptor` value is a `Record`. Each field is an attribute and the value of the field is the value of that attribute. Any field can be present or absent.

There are 3 kinds of `Property Descriptor`s, *data `Property Descriptor`*, *accessor `Property Descriptor`* and *generic `Property Descriptor`*.

A data `Property Descriptor` has a `[[Value]]` or `[[Writable]]` (or both) field.
An accessor `Property Descriptor` has a `[[Get]]` or `[[Set]]` (or both) field.

In both cases the other 2 fields are `[[Enumerable]]` and `[[Configurable]]`.

A generic `Property Descriptor` is neither a data `Property Descriptor` nor an accessor `Property Descriptor` (in this case it is not fully-populated, i.e. only has the `[[Writable]]` and/or `[[Configurable]]` fields).

The abstract operations on a `Property Descriptor` are: `IsAccessorDescriptor`, `IsDataDescriptor`, `IsGenericDescriptor`, `FromPropertyDescriptor`, `ToPropertyDescriptor`, `CompletePropertyDescriptor`.

#### `Data Block` type

The `Data Block` type is used to describe a *distinct and mutable* sequence of byte-sized (8 bit) numeric values.

A `Data Block` value is created with a fix number of bytes that each has the initial value of `0`. An array-like syntax is used to access the individual bytes of a `Data Block` value. Each byte is an element in the `Data Block`.

A `Data Block` value in memory that can be concurrently referenced from multiple places, is called a *shared `Data Block`*. A shared `Data Block` has an *identity* used for equality testing.

The abstract operations defined on `Data Block`s values are: `CreateByteDataBlock`, `CreateSharedByteDataBlock`, `CopyDataBlockBytes`.
