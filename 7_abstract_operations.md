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
