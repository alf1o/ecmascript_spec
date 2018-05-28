# Abstract Operations

Abstract operations are algorithms which aren't part of the ES language, but are used to better describe its inner working.

# Type conversion

*The ES language automatically performs type conversion when needed.*

Conversion abstract operations are used to describe the way this type conversion happens. These abstract operations accept any ES language value, but no specification values.

## ToPrimitive(*input [, PreferredType]*)

`ToPrimitive` accepts an `input` and an optional `PreferredType` as arguments.
This abstract operation converts `input` into a non-`object` type. If an object can convert to more than one primitive type, then the `PreferredType` can be used to specify which.

When `ToPrimitive` is called without the `PreferredType` hint, then the default is set to `number`. But the object we are trying to convert might define its own `@@toPrimitive` method, thus overriding the default. For example, `Date` and `Symbol` define their own `@@toPrimitive`.

- `ToPrimitive` first checks that `input` is actually an `object`. If it is a primitive value, then the operation immediately returns with that value.
- Then `ToPrimitive` checks if a `PreferredType` was passed. If that's not the case the default hint is set to `"default"`.
- `ToPrimitive` checks if the `input` has its own `@@toPrimitive` method defined. If that's the case then that method is called passing in the `input` and the hint.
  - If the result is a primitive value `ToPrimitive` returns that value.
  - If not, then `ToPrimitive` throws a `TypeError`.
- If the `input` doesn't have its `@@toPrimitive` then the hint is set to `"number"` and  the `OrdinaryToPrimitive` abstract operation is called passing in the `input` and the hint.
- The result of this operation is returned.

## OrdinaryToPrimitive(*O, hint*)

This abstract operation will check if the `O` object has the `toString` and `valueOf` method defined (or if they are available in its `[[Prototype]]` chain) and based on the `hint` will call either of them, leaving the other as fallback.

The `hint` can only be `"number"` or `"string"`. In case of `"number"` first `valueOf` is called on the `O`, and if it doesn't exist, or if it fails in returning a primitive value, then `toString` is called.
If instead the `hint` is `"string"` the opposite happens. First `OrdinaryToPrimitive` tries to invoke `toString`, but if it doesn't exist or if it fails in returning a primitive, then `valueOf` is called.
In case both methods fail `OrdinaryToPrimitive` will throw a `TypeError`.

## ToBoolean(*argument*)

The `ToBoolean` abstract operation converts `argument` to a `boolean` value.

ES defines a "falsy-list". Whatever is on this list is coerced to `false`. Everything else is coerced to `true`.

The "falsy-list" contains: `"", NaN, 0 (both +0 and -0), null, undefined, false`.

That's it. All other values, included things like empty objects, empty arrays, ..., are coerced to `false` by `ToBoolean`.

## ToNumber(*argument*)

This operation will convert a value to the `number` type.

Based on the type of `argument` one of these happens:
- `undefined` -> `NaN`
- `null` -> `+0`
- `number` -> `argument` (no conversion applied)
- `boolean`
  - `true` -> `1`
  - `false` -> `+0`
- `symbol` -> `TypeError`

**NB:** a `symbol` value cannot be coerced to a `number`.

For a `string` value a grammar and conversion algorithm is applied. In case it fails the result is `NaN`.

For an `object` value first `ToPrimitive` is called passing in the object and `number` as `PreferredType`. The result of this operation is then converted to `number` (in case `ToPrimitive` fails, `ToNumber` also fails).

## ToInteger(*argument*)

This abstract operation converts `argument` to an integer `number`.

It first calls `ToNumber` on the `argument`.
Based on the result of that `ToInteger` will return:
- `NaN` -> `+0`
- `+0/-0/+Infinity/-Infinity` -> `+0/-0/+Infinity/-Infinity` (no further transformation applied)

To any other value returned by `ToNumber`, `ToInteger` will apply `floor(abs(value))` and return that with the original `value` sign.

## ToString(*argument*)

The `ToString` abstract operation converts any value to a `string` value.

For primitive values, ES defines a "natural" stringification.
Based on the type of `argument` `ToString` returns:
- `undefined` -> `'undefined'`
- `null` -> `'null'`
- `boolean`
  - `true` -> `'true'`
  - `false` -> `'false'`
- `string` -> `argument` (no conversion applied)
- `symbol` -> `TypeError`

For an `object` value `ToString` first calls `ToPrimitive` passing in the `argument` and `string` as `PreferredType`. Then the result of that is converted to a `string` value.

In case of a `number` value, `ToString` calls the `NumberToString` abstract operation returning its result.

## ToObject(*argument*)

`ToObject` converts a value of any type to an `object` value.

Based on the type of `argument` `ToObject` returns a new object of the sub-type corresponding to `argument` type and fills an internal slot of this object with `argument` itself.

- `undefined` -> `TypeError`
- `null` -> `TypeError`
- `boolean` -> `[object Boolean]` with the internal slot `[[BooleanData]]`
- `number` -> `[object Number]` with the internal slot `[[NumberData]]`
- `string` -> `[object String]` with the internal slot `[[StringData]]`
- `symbol` -> `[object Symbol]` with the internal slot `[[SymbolData]]`
- `object` -> `argument` (no conversion applied)

## ToPropertyKey(*argument*)

This abstract operation converts `argument` into a value that can be used an property key on an object.

- First call `ToPrimitive` passing in the `argument` and `string` as `PreferredType`.
- If the result has type of `symbol` then return it.
- Otherwise call `ToString` on it and return that result instead.

# Testing and comparison operators

## RequireObjectCoercible(*argument*)

The `RequireObjectCoercible` abstract operation throws a `TypeError` if the `argument` cannot be converted to `object` by using the `ToObject` abstract operation. Else it returns the `argument` as-is.

This means that a value of type `boolean`, `string`, `number`, `symbol` or `object` will be returned.
While a value of type `null` or `undefined` will throw an error.

## IsArray(*argument*)

This abstract operation returns `true` if the `argument` is an array, else it returns `false`. In case the `argument` is a `Proxy` object, if its `[[ProxyHandler]]` internal property is `null`, then `IsArray` throws a `TypeError`.

`IsArray` follows these steps:
- First check that the type of the `argument` is actually `object`. In case it's not return `false`.
- If it's an `object` and is actually an array then return `true`.
- If it's a Proxy object then
  - if it has a `[[ProxyHandler]]` which is **null** then throw a `TypeError`
  - else get the target value from the Proxy internal property `[[ProxyTarget]]`
  - return the result of calling `IsArray` on the target
- Else if the object is not an array nor a Proxy, then return `false`

## IsCallable(*argument*)

This abstract operation only accepts an `argument` which is an ES language value. It returns `true` if the `argument` is a function (callable object).

`IsCallable` abstract operation:
- First check if the `argument` is of type `object`, if it's not return `false`
- If it's an object and it has an internal `[[Call]]` method, then return `true`
- Else return `false`

## IsConstructor(*argument*)

This abstract operation only accepts an `argument` which is an ES language value and it checks if this `argument` is a function with a `[[Construct]]` internal method.

`IsConstructor` abstract operation:
- First check if the type of `argument` is `object`, if not return `false`
- If it's an object an it has the `[[Construct]]` internal method return `true`
- Else return `false`

## IsExtensible(*O*)

This abstract operation accepts an `object` as input and returns a `boolean` saying whether or not new properties can be added to this input.

`IsExtensible` will simply call the `[[IsExtensible]]` essential internal method on the `O` object and return its result.

## IsInteger(*argument*)

This abstract operation checks if the `argument` is a finite integer `number`.

`IsInteger` abstract operation:
- First check if the type of `argument` is `number`, if it's not return `false`
- If it's a number, but it is `NaN` or `+Infinity` or `-Infinity` return `false`
- Else for every other number if `floor(abs(argument))` is different from `abs(argument)` return `false`
- Else return `true`

## IsPropertyKey(*argument*)

This abstract operation only accepts an `argument` which is an ES language value and it checks if `argument` can be used as a property key for an object.

`IsPropertyKey` checks if the type of `argument` is either `string` or `symbol`. If it is then `IsPropertyKey` returns `true`. Else it returns `false`.

## IsRegExp(*argument*)

This abstract operation checks if the `argument` is a RegExp object.

`IsRegExp` abstract operation:
- First check if the type of `argument` is `object`, else return `false`
- If it is an object, retrieve its `@@match` method by using `Get` abstract operation passing in the `argument` and `@@match`
- If the matcher result is not **undefined**, then return the result of calling the `ToBoolean` abstract operation on the matcher
- Else if the matcher is **undefined** then if the `argument` has a `[[RegExpMatcher]]` internal slot return `true`
- Else return `false`

## IsStringPrefix(*p, q*)

This abstract operation checks if the string `p` is a prefix of the string `q` by checking if `q` can be the string concatenation of `p` and another string `r`.

## SameValue(*x, y*)

This comparison abstract operation takes in two ES language values and checks if they are actually the same value.

This abstract operation differs from the `Strict Equality Comparison` algorithm in the way it treats `-0`, `+0` and `NaN`.

`SameValue` abstract operation:
- First check that `x` and `y` have the same type, else return `false`
- If the type is `number` then
  - if both `x` and `y` are `NaN`, then return `true`
  - if `x` is `-0` and `y` is `+0` return `false`
  - if `x` is `+0` and `y` is `-0` return `false`
  - if `x` is the same `number` value than `y`, then return `true`
  - else return `false`
- Else if the type is not `number` return the result of calling the `SameValueNonNumber` abstract operation passing in `x` and `y`

## SameValueZero(*x, y*)

This comparison abstract operation takes in two ES language values and checks if they are the same value.

It differs from `SameValue` in the way it treats `-0` and `+0`.

`SameValueZero` abstract operation:
- First check that `x` and `y` have the same type, else return `false`
- If the type is `number` then
  - if both `x` and `y` are `NaN`, then return `true`
  - if `x` is `-0` and `y` is `+0` return `true`
  - if `x` is `+0` and `y` is `-0` return `true`
  - if `x` is the same `number` value than `y`, then return `true`
  - else return `false`
- Else if the type is not `number` return the result of calling the `SameValueNonNumber` abstract operation passing in `x` and `y`

## SameValueNonNumber(*x, y*)

This comparison abstract operation takes in two ES language values of any type but `number` and checks if they are actually the same value.

`SameValueNonNumber` is always called with two values of the same type and that type is not `number`, so this abstract operation doesn't need to check for that.

`SameValueNonNumber` abstract operation:
- If type of `x` (and thus type of `y`) is `undefined`, then return `true`
- If type of `x` is `null`, then return `true`
- If type of `x` is `string`
  - if `x` and `y` are the exact same sequence of code units, then return `true`
  - else return `false`
- If type of `x` is `boolean`
  - if `x` and `y` are both `true` or both `false`, then return `true`
  - else return `false`
- If type of `x` is `symbol`
  - if `x` and `y` are the same `symbol` value then return `true`
  - else return `false`
- Else, type of `x` is `object`
  - if `x` and `y` are the same `object` vale then return `true`
  - else return `false`

## Abstract Relational Comparison

This algorithm is used in evaluation of expressions like `x < y`, where both `x` and `y` are values. It returns `true` if `x` is indeed lower than `y` else it returns `false`.

The `Abstract Relational Comparison` algorithm also accepts a `boolean` flag called LeftFirst. This flag is used to control the order in which operations with visible side-effects are performed upon `x` and `y`. ES specifies left-to-right evaluation for expressions. If LeftFirst is `true` (default) then `x` corresponds to an expression that happens to the left of `y`.
```
For example, the algorithm will turn both `x` and `y` into a primitive value. The `ToPrimitive` abstract operation might have side-effects (i.e. throwing an error), thus it will be applied first on `x` or first on `y` based on the value of LeftFirst.(???)

or

If LeftFirst is `true` then `x` was evaluated by an expression happening to the left of the expression that computed `y`.

or

This means that `x > y` is passed to this algorithm as `y < x`.
```

The result of the `Abstract Relational Comparison` algorithm can also be **undefined** in case at least one of the arguments is `NaN`.

The `Abstract Relational Comparison` algorithm allows coercion, so it first converts both values `x` and `y` to a primitive value. If **both** primitives are of type `string` then the algorithm performs a lexicographic comparison.

If at least one is not a `string` value, then the algorithm converts them both to `number` and then compares the results.

The `Abstract Relational Comparison` algorithm ignores signed zeroes, so it treats `+0` and `-0` as the same value. This means that both `+0 < -0` and `-0 < +0` return `false`.

## Abstract Equality Comparison

This algorithm is used in evaluation of expressions like `x == y`, where both `x` and `y` are values.

The `Abstract Equality Comparison` algorithm allows coercion. It returns either `true` or `false`.

If `x` and `y` have the same type, the algorithm delegates the comparison to the `Strict Equality Algorithm` and returns that result.
Else the `Abstract Equality Comparison` algorithm recursively coerces `x` and `y` until they are of the same type, then compares them.

The rules for the coercion are:
- If type of `x` is `undefined`
  - if type of `y` is `null`, then return `true`
  - else return `false`
- If type of `x` is `null`
  - if type of `y` is `undefined`, then return `true`
  - else return `false`
- If type of `x` is `number` and type of `y` is `string` then call `ToNumber(y)`
- If type of `x` is `string` and type of `y` is `number` then call `ToNumber(x)`
- If type of `x` is `boolean` then call `ToNumber(x)`
- If type of `y` is `boolean` then call `ToNumber(y)`
- If type of `x` is one of `string`, `number`, `symbol` and type of `y` is `object` then call `ToPrimitive(y)`
- If type of `x` is `object` and type of `y` is one of `string`, `number`, `symbol` then call `ToPrimitive(x)`

So, if we have a `boolean` value, that value always goes to `number` independently from the type of the other operand.
If we have a `string` and a `number` is always the `string` that goes to `number`.
If we have an `object` and a primitive value is always the `object` that goes to primitive.
`null` and `undefined` are considered equal.

## Strict Equality Comparison

This algorithm is used for the evaluation of expressions like `x === y`, where `x` and `y` are two values.

The `Strict Equality Comparison` algorithm doesn't allow type conversion, which means it returns `false` every time `x` and `y` have different type.

This algorithm handles directly the comparison of `x` and `y` if they have type `number`. The comparison of values of any other type is delegated to the `SameValueNonNumber` algorithm.

The `Strict Equality Comparison` algorithm treats any `NaN` value different from any other `NaN` value (even itself). Also it doesn't consider signed zeroes, so both `-0 === +0` and `+0 === -0` return `true`.

# Abstract operations on objects

## Get(*O, P*)

The `Get` abstract operation takes in an object `O` and a valid property key `P` and retrieves the value of `P` on `O`.

This operation calls the object `[[Get]]` essential internal method passing in `P` and `O` itself, which will be used as context in case `P` is a getter.

## GetV(*V, P*)

This abstract operation takes in any ES language value `V` and a valid property key `P` and retrieves the value of the property `P` on `V`. If `V` is not an `object` value, then it is first wrapped within an object of the subtype corresponding to `V` type by the `ToObject` abstract operation. Then the `[[Get]]` essential internal method is called on this object passing in `P` and `V`, which will be used as context in case `P` is a getter.

## Set(*O, P, V, Throw*)

The `Set` abstract operation is used to set the value of a property on an object. It will return either `true` or `false` or `throw` an error.

`Set` takes in an object `O`, a valid property key `P`, any ES language value `V` and a `boolean` flag called `Throw`.

`Set` abstract operation:
- First call the `[[Set]]` essential internal method on the object, passing in `P`, `V` and `O` itself, which will be used as context in case `P` is a setter.
`[[Set]]` internal method returns `true` if the operation succeeded, else it returns `false`.
- If `[[Set]]` returns `false` and `Throw` is `true`, then throw a `TypeError`.
- Else return the result of `[[Set]]`

## CreateDataProperty(*O, P, V*)

This abstract operation is used to create a new own property of the object `O`. `P` is a valid property key and `V` is the value to be assigned to `P`.

`CreateDataProperty` abstract operation:
- First create a new `PropertyDescriptor` with `[[Value]]` being `V` and all the other fields set to `true`.
- Then call the `[[DefineOwnProperty]]` essential internal method on `O` passing in `P` and the `PropertyDescriptor`. `[[DefineOwnProperty]]` returns `true` if the operation succeeded, else returns `false`.
- Return the result of `[[DefineOwnProperty]]`

`CreateDataProperty` creates a new property with the same default fields set by the assignment operator.

If the property `P` already exists on `O` and it is not configurable, then `[[DefineOwnProperty]]` returns `false`.

If `O` is not extensible, then `[[DefineOwnProperty]]` also returns `false`.

## CreateMethodProperty(*O, P, V*)

This abstract operation is used to create a new own property on the object `O`.
`P` is a valid property key and `V` is the value to be assigned to `P`.

The `CreateMethodProperty` abstract operation works just like `CreateDataProperty`, the difference is that the `PropertyDescriptor` this time has the `[[Enumerable]]` field set to `false`.

`CreateMethodProperty` creates a new property with the same default fields used for built-in methods and for methods defined using class declaration syntax.

## CreateDataPropertyOrThrow(*O, P, V*)

The `CreateDataPropertyOrThrow` abstract operation is used to set a new own property on an object, but if the operation fails it throws an error.

`CreateDataPropertyOrThrow` first calls `CreateDataProperty`, then if it returns `false`, `CreateDataPropertyOrThrow` throws a `TypeError`.

## DefinePropertyOrThrow(*O, P, desc*)

This abstract operation calls the `[[DefineOwnProperty]]` essential internal method on the object `O`, passing in a valid property key `P` and a `PropertyDescriptor` `desc`. If `[[DefineOwnProperty]]` returns `false`, then `DefinePropertyOrThrow` throws a `TypeError`

## DeletePropertyOrThrow(*O, P*)

This abstract operation calls the `[[Delete]]` essential internal method on the object `O` passing in the valid property key `P`. `[[Delete]]` returns `true` if the operation was successful, else it returns `false` (i.e. `P` is a non-configurable property).
In case `[[Delete]]` returns `false`, `DeletePropertyOrThrow` throws a `TypeError`.

## GetMehod(*V, P*)

This abstract operation is used to retrieve a property on a value, and that property is expected to be a function.

`GetMehod` takes in any ES language value `V` and a valid property key `P` and returns `undefined` or the function or `throw`s a `TypeError`.

The `GetMehod` abstract operation:
- First call `GetV` passing in the value `V` and the property key `P`. `GetV` accepts any ES language value and wraps it in an object in case it's a primitive type.
- If `GetV` returns `undefined` or `null`, then return `undefined`
- Else check if the result is a function (i.e. has a `[[Call]]` internal method)
  - if it doesn't, throw a `TypeError`
  - else return that function

## HasProperty(*O, P*)

This abstract operation is called with an object `O` and a valid property key `P` and it checks if `P` is a property on the object `O` (either own or inherited).

`HasProperty` calls the `[[HasProperty]]` essential internal method on the object `O` passing in `P` and returns its result (either `true` or `false`).

## HasOwnProperty(*O, P*)

This abstract operation takes in an object `O` and a valid property key `P` and it checks if `P` is an own property of `O`.

`HasOwnProperty` calls the `[[GetOwnProperty]]` essential internal method on the object `O` passing in the property key `P`. This method returns a `PropertyDescriptor` or `undefined` if `P` is not on `O`. `HasOwnProperty` returns `true` and `false` respectively.

## Call(*F, V[, argumentsList]*)

The `Call` abstract operation is used to invoke the `[[Call]]` essential internal method on a function object.

`Call` takes in a value `F` supposed to be the function, an ES language value `V` which is used by `[[Call]]` as value of `this` for this function invocation, and optionally a list of arguments, which are the arguments with which the function is invoked.

The `Call` abstract operation:
- First check if `argumentsList` is present, if not set it to be a new empty `List`
- Then check if `F` is actually a function (i.e. has an internal `[[Call]]` method) and if it's not throw a `TypeError`
- Else, execute the `[[Call]]` internal method on `F` passing in `V` and the `argumentsList` and return its result

## Construct(*F[, argumentsList[, newTarget]]*)

This abstract operation takes in a function object `F` which can be called as a constructor function, and optionally an `argumentsList`, which is the list of arguments with which `F` was invoked, and a `newTarget` which is the value to which the `new` keyword was applied.

**NB:** by using `@@species` we can change the target of the `new` keyword.

`Construct` calls the `[[Construct]]` internal method on `F` passing in the `argumentsList` and `newTarget`. This method builds a new object and sets the context of `F` to be that object. It also returns this object in case `F` doesn't explicitly return anything else.

The `Construct` abstract operation:
- First check if `newTarget` was passed and if not set `newTarget` to be `F`
- Check if `argumentsList` was passed, else set it to be an empty `List`
- Return the result of calling the `[[Construct]]` internal method on `F` passing in the `argumentsList` and `newTarget`.

## SetIntegrityLevel(*O, level*)

This abstract operation is used to set the level of immutability of an object. It takes in an object `O` and a `level` which can be either `"sealed"` or `"frozen"`.

`SetIntegrityLevel` first tries to make the object non-extensible. If this operation fails, then `SetIntegrityLevel` returns `false`, else it either succeeds and returns `true` or fails with a `TypeError`.

The `SetIntegrityLevel` abstract operation:
- First call the `[[PreventExtensions]]` essential internal method on the object `O`.
- If this method returns `false`, then return `false`.
- Else retrieve all the keys of the own properties of the object `O` by calling the `[[OwnPropertyKeys]]` essential internal method on `O`. This method returns a `List` of the keys.
- If `level` is `"sealed"` then for each key in keys call the `DefinePropertyOrThrow` abstract operation passing in the object `O`, the key and the `PropertyDescriptor{ [[Configurable]]: false }`.
- Else `level` is `"frozen"` then for each key in keys
  - call the `[[GetOwnProperty]]` essential internal method on the object `O` passing in the key
  - if the result is an accessor `PropertyDescriptor` then call the abstract operation `DefinePropertyOrThrow` passing in the object `O`, the key and the `PropertyDescriptor{ [[Configurable]]: false }`
  - else the result is either a data or generic `PropertyDescriptor` and call `DefinePropertyOrThrow` passing in `O`, the key and `PropertyDescriptor{ [[Configurable]]: false, [[Writable]]: false }`
- Return `true`

If `DefinePropertyOrThrow` throws a `TypeError`, then `SetIntegrityLevel` also throws that same error.

## TestIntegrityLevel(*O, level*)

This abstract operation takes in an object `O` and a `level` (either `"sealed"` or `"frozen"`) and checks if `O` has that immutability level. `TestIntegrityLevel` retrieves all the own property keys of `O` and for each key, if the `level` is `"sealed"`, checks that the `[[Configurable]]` attribute is `false`. If the `level` is `"frozen"`, it additionally checks if the `[[Writable]]` attribute is `false`.

The `TestIntegrityLevel` abstract operation:
- First check if `O` is extensible, and if it is, then return `false` (no property is checked)
- If `O` is not extensible, then retrieve a `List` of the keys of all its own properties by calling the essential internal method `[[OwnPropertyKeys]]`
- For each key in the `List`
  - get the property descriptor by calling the essential internal method `[[GetOwnProperty]]` passing in key
  - if the `[[Configurable]]` field is `true`, the return `false`
  - else if `level` is `"frozen"` and if the property at this key is a data descriptor (`IsDataDescriptor` returns `true`) then if the `[[Writable]]` field is `true`, then return `false`
- Return `true`

## CreateArrayFromList(*elements*)

This abstract operation takes in a `List` of ES language values and uses it to generate an array `object.`

The `CreateArrayFromList` abstract operation:
- First create an array of `length` set to `0` by calling the `ArrayCreate` abstract operation
- Then, set `n` to be `0` and for each element in `elements`
  - execute `CreateDataProperty` passing in the array, `ToString(n)` and the element
  - increment `n` by `1`
- Return the array

## CreateListFromArrayLike(*obj[, elementTypes]*)

# Abstract operations on iterator objects

## GetIterator(*obj[, hint[, method]]*)

This abstract operation tries to retrieve a `Record` representing the iterator of an object `obj`. The `method` argument is the method on `obj` that is used to retrieve such iterator. The `hint` is either `"sync"` or `"async"` and if `method` is not passed, then `hint` is used to retrieve either the `@@iterator` or the `@@asyncIterator` method on the `obj`.

The `Record` returned has 3 fields: `[[Iterator]]`, with the value of the iterator object on `obj`, `[[NextMethod]]`, with the value of the `next` method of this iterator, `[[Done]]`, a `boolean` representing if the iterator is done running.

The `GetIterator` abstract operation:
- If `hint` is not present set it to `"sync"`
- If `method` is not present then
  - if `hint` is `"async"`
    - try retrieve the `@@asyncIterator` method on `obj` (by using `GetMehod`) and assign it to `method`
    - if `obj` doesn't have an `@@asyncIterator` method
      - then try retrieve the `@@iterator` method
      - then call `GetIterator` on `obj`, passing `"sync"` as `hint` and `@@iterator` as method
      - create an async iterator `Record` from the result, and return it
  - if instead `hint` is `"sync"` then try retrieve the `@@iterator` method
- Try retrieve the iterator by calling `method` on the `obj`
- If the iterator is not an `object`, then throw a `TypeError`
- Else, get the `next` method from the iterator (by calling `GetV`)
- Generate the `Record{ [[Iterator]]: iterator, [[NextMethod]]: nextMethod, [[Done]]: false }` and return it

## IteratorNext(*iteratorRecord[, value]*)

This abstract operation calls the `next` method on the iterator represented by `iteratorRecord`. The result of the call, should be an `object`, which is then returned. If `value` is present, it is passed as argument in the call of `next`.

The `IteratorNext` abstract operation:
- If `value` is not present, then call the `next` method of the iterator (`iteratorRecord.[[NextMehod]]`) passing an empty `List` of arguments
- Else, call the `next` method passing a `List` containing the `value`
- If the result of the call is not an `object`, then throw a `TypeError`
- Else return that result

## IteratorComplete(*iterResult*)

This abstract operation takes in an `object` value, which is the result of the call of the `next` method on an iterator object, and returns the value of the `done` property of this object converted to `boolean` by the `ToBoolean` abstract operation (`ToBoolean(Get(iterResult, "done"))`).

## IteratorValue(*iterResult*)

This abstract operation takes is an object, which is the result of calling `next` on an iterator, and returns the value of the `value` property of this object.

## IteratorStep(*iteratorRecord*)

This abstract operation takes in the `Record` of an iterator object and checks if there are more iterations to do. It calls the `next` method on the iterator and checks if `done` on the result is `true`. If there are no more iterations to perform, `IteratorStep` returns `false`, else it returns the result object.

The `IteratorStep` abstract operation:
- First call the `IteratorNext` abstract operation passing the `iteratorRecord`
- Then call the `IteratorComplete` passing in the result
- If done is `true`, then return `false`
- Else return the result object

## IteratorClose(*iteratorRecord, completion*)

This abstract operation is called with an iterator `Record` and a `CompletionRecord` and terminates the iterator right away rather then moving it to its next state.

The `IteratorClose` abstract operation:
- Retrieve the iterator object from the `[[Iterator]]` field of `iteratorRecord`
- Try get the `return` method from the iterator
- If there isn't, return the `completion` `CompletionRecord`
- Else call `return` with an empty `List` of arguments, using the iterator as context and save the result in innerResult
- If `completion` has a `[[Type]]` field of `throw`, then return `completion`
- Else if innerResult has a `[[Type]]` of `throw`, then return innerResult
- Else, if innerResult has a `[[Value]]` field value which is not an `object`, then throw a `TypeError`
- Else, return `completion`

## AsyncIteratorClose(*iteratorRecord, completion*)

This abstract operation is used to terminate an async iterator.

The `AsyncIteratorClose` abstract operation:
- First get the iterator from the `[[Iterator]]` field of `iteratorRecord`
- Try get the `return` method from the iterator
- If there isn't, return `completion`
- Else, call `return` passing the iterator as context and an empty `List` of arguments and store the result in innerResult
- If innerResult has a `[[Type]]` field of `normal`, then set innerResult to be the result of calling the `Await` abstract operation passing in `innerResult.[[Value]]`
- If `completion` has a `[[Type]]` of `throw`, then return `completion`
- Else if innerResult has a `[[Type]]` of `throw`, then return innerResult
- Else if the value of the `[[Value]]` field of innerResult is not an `object`, then throw a `TypeError`
- Else return `completion`

## CreateIterResultObject(*value, done*)

This abstract operation takes in a any ES language value as `value`, and a `done` argument which is a `boolean`, and creates a new object that can be used as result of a call to an iterator `next` method.

The `CreateIterResultObject` abstract operation:
- Create an ordinary object by calling `ObjectCreate(%ObjectPrototype%)`
- Create a data property called `"value"` with the value of `value`
- Create a data property called `"done"` with the value of `done`
- Return the object

## CreateListIteratorRecord(*list*)

This abstract operation is called with a `List` argument and returns a `Record` representing an iterator object. Calls of the `next` method of this iterator will return the values inside `list`.

The `CreateListIteratorRecord` abstract operation:
- Create a new ordinary object with 2 empty internal slots named `[[IteratedList]]` and `[[ListIteratorNextIndex]]` by calling `ObjectCreate` passing in `%IteratorPrototype%` and a `List` with the internal slots
- Set the value of `iterator.[[IteratedList]]` to be `list`
- Set the value of `iterator.[[ListIteratorNextIndex]]` to be `0`
- Define **steps** as being the algorithm steps of a standard iterator `next` method
- Create a `next` method by calling `CreateBuiltInFunction` passing in **steps**
- Build an iterator `Record{ [[Iterator]]: iterator, [[NextMehod]]: next, [[Done]]: false }` and return it

## ListIterator next()

This is a standard built-in function. Its algorithm is the one used by `CreateListIteratorRecord` to create the `next` method for the iterator object.
It returns an object with a `value` and a `done` property.

- Let `O` be an object used as the value of `this`
- Define **list** as being the value of the internal slot `[[IteratedList]]` on `O`
- Define **index** as being the value of the internal slot `[[ListIteratorNextIndex]]` on `O`
- Define **len** as being the number of elements in `list`
- If **index** is greater or equal than **len**, then create a new object with a `value` property of `undefined` and a `done` property of `true` (by calling `CreateIterResultObject`) and return it
- Else set `O.[[ListIteratorNextIndex]]` to be `index + 1`
- Create a new object with a `value` property of `list[index]` and a `done` property of `false` and return it
