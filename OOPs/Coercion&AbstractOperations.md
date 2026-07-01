# JavaScript Type Coercion and Abstract Operations

## 1. Overview

JavaScript performs both explicit and implicit type conversion. This flexibility is powerful but also a common source of confusing behavior, since conversions between types follow specific internal rules defined in the ECMAScript specification.

JavaScript defines the following types:

```
Undefined, Null, Boolean, Number, BigInt, String, Symbol, Object
```

Older versions of the spec (pre-ES2020, e.g. the 10.0/ES2019 edition) list only seven types, since `BigInt` was introduced in ES2020 as an 8th primitive type.

## 2. Abstract Operations

Abstract operations are not part of the ECMAScript language itself. They are internal algorithms defined purely to describe behavior inside the specification. Developers cannot call them directly (e.g. there is no `ToNumber()` global function to invoke) - they exist to define how the engine performs conversions internally, such as during type coercion.

Common abstract operations include `ToBoolean`, `ToNumber`, `ToString`, and `ToPrimitive`.

## 3. Implicit vs Explicit Conversion

- **Explicit conversion**: the developer manually converts a value's type.
  ```js
  Number("5")   // 5
  String(5)     // "5"
  Boolean(1)    // true
  ```

- **Implicit conversion (coercion)**: the engine automatically converts a value's type as part of evaluating an expression.
  ```js
  "5" - 1     // 4
  !!value
  1 + "2"     // "12"
  ```

Coercion refers specifically to this automatic, implicit conversion.

## 4. ToBoolean

The `ToBoolean` abstract operation converts a value to a Boolean. The logical NOT operator (`!`) internally relies on `ToBoolean`: it first coerces its operand to a Boolean, then inverts it.

| Argument Type | Result |
|---|---|
| Undefined | false |
| Null | false |
| Boolean | returns the argument unchanged |
| Number | false if +0, -0, or NaN; otherwise true |
| String | false if the string is empty; otherwise true |
| Symbol | true |
| Object | true |

### Examples

```js
!(3)        // false   -> 3 is truthy, NOT flips it to false
!("false")  // false   -> "false" is a non-empty string, so it is truthy; NOT flips it to false
!{}         // false   -> objects are always truthy
```

A key point with strings: the *content* of the string does not matter, only whether it is empty. `"false"`, `"0"`, and `" "` are all non-empty strings and therefore truthy.

`ToBoolean` is also used implicitly by logical `&&` and `||`. These operators coerce operands to decide which one to return, but they return the operand itself rather than a strict boolean:

```js
1 && "hi"   // "hi"
0 || "hi"   // "hi"
```

## 5. ToNumber

The `ToNumber` abstract operation converts a value to a Number.

| Argument Type | Result |
|---|---|
| Undefined | NaN |
| Null | +0 |
| Boolean | 1 if true, +0 if false |
| Number | returned unchanged |
| String | parsed according to the numeric grammar; non-numeric strings become NaN |
| Symbol | throws a TypeError |
| Object | call `ToPrimitive(argument, hint Number)`, then apply `ToNumber` to the result |

Because the Object case can call back into `ToNumber` after `ToPrimitive` resolves, `ToNumber` behaves recursively for objects.

### String-to-number conversion

Each numeric string converts independently to its numeric value:

```js
Number("3")   // 3
Number("1")   // 1
Number("2")   // 2
```

There is no concatenation behavior in `ToNumber` itself - a result like `"312"` only arises if strings are concatenated first (e.g. `"3" + "1" + "2"`) and *then* converted.

## 6. Operator Behavior

### Subtraction (`a - b`)

Both operands are converted to Number, then subtracted:

```js
1 - "2"   // ToNumber(1) - ToNumber("2") -> 1 - 2 -> -1
```

### Addition (`a + b`)

1. `ToPrimitive` is applied to both operands.
2. If either result is a String, both operands are converted with `ToString` and concatenated.
3. If neither is a String, both are converted with `ToNumber` and added numerically.

```js
1 + "2"   // "12"  (string concatenation, since one operand is a string)
1 - "2"   // -1    (numeric subtraction)
```

Note that `1 + "2"` produces the string `"12"`, not the number `12`.

## 7. Primitive vs Non-Primitive Values

- **Primitive values** are atomic - they cannot be broken down further. Examples: Boolean, Number, String, Symbol, BigInt, Undefined, Null.
- **Non-primitive values (objects)** are composite structures built out of primitive values (and other objects) as properties. They are compared and stored by reference rather than by value.

## 8. Equality Comparisons: `==` vs `===`

### Abstract Equality Comparison (`==`)

If the operand types differ, `==` performs implicit conversions before comparing. The algorithm:

1. If `Type(x)` is the same as `Type(y)`, return the result of `x === y`.
2. If `x` is `null` and `y` is `undefined`, return `true`.
3. If `x` is `undefined` and `y` is `null`, return `true`.
4. If `Type(x)` is Number and `Type(y)` is String, convert `y` to a Number and compare.
5. If `Type(x)` is String and `Type(y)` is Number, convert `x` to a Number and compare.
6. If `Type(x)` is Boolean, return the result of `ToNumber(x) == y`.
7. If `Type(y)` is Boolean, return the result of `x == ToNumber(y)`.
8. If `Type(x)` is String, Number, or Symbol and `Type(y)` is Object, return the result of `x == ToPrimitive(y)`.
9. If `Type(x)` is Object and `Type(y)` is String, Number, or Symbol, return the result of `ToPrimitive(x) == y`.
10. Otherwise, return `false`.

(Later spec editions insert additional steps to handle `BigInt` comparisons between the existing steps above.)

### Strict Equality Comparison (`===`)

1. If `Type(x)` is different from `Type(y)`, return `false`.
2. If `Type(x)` is Number:
   - If `x` is `NaN`, return `false`.
   - If `y` is `NaN`, return `false`.
   - If `x` and `y` are the same Number value, return `true`.
   - If `x` is `+0` and `y` is `-0`, return `true`.
   - If `x` is `-0` and `y` is `+0`, return `true`.
   - Otherwise, return `false`.
3. Otherwise, return the result of `SameValueNonNumber(x, y)`.

Unlike `==`, `===` never performs implicit type conversion - if the types differ, the comparison is immediately `false`.

## 9. Key Takeaways

- Coercion is implicit conversion; it happens automatically during operations like `+`, `-`, `!`, and `==`.
- `ToBoolean` treats every value as truthy except: `false`, `0`, `-0`, `NaN`, `""`, `null`, and `undefined`.
- `+` prefers string concatenation if either operand is (or becomes) a string; all other arithmetic operators coerce to Number.
- `ToNumber` on an object goes through `ToPrimitive` first, making it effectively recursive.
- `==` allows type coercion between operands; `===` does not.