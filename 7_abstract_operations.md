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
