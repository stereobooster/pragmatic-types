# Type system vs null

> Null References: The Billion Dollar Mistake
>
> -- Tony Hoar, the inventor of a null reference. [See his talk](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)

## What is null and why you should care?

Null represents an absence of value. For example, if you try to get value from an array (vector), there is a chance that value will be missing (if the array is empty). What system can do in that case?

- throw exception (or return it, like they do in Go)
- return value which represents an absence of value - null value

Also, null values used for uninitialized values, you need those to construct cyclic structures.

## What is the issue

The main issue is when a null value is considered to be part of all types e.g. null is a valid number or valid string, but same time you can not apply any of operations for the given type to a null value.

```js
"" + undefined
"undefined"

1 * undefined
NaN
```

## Null value and JavaScript

Tony Hoar said that null reference is the billion dollar mistake. JavaScript doubles it by introducing two null values: `null` and `undefined`. I'm not sure why, I believe there is some historical reason, probably null was introduced to brand JS as close to Java as possible (the same as name choice).

```js
null === undefined
false
null == undefined
true
1 * undefined
NaN
1 * null
0
```

## Type-based solution

Flow and other modern type systems don't consider `null` (or `undefined`) as part of any "basic" type:

Flow:

```ts
let x:number = undefined;
                  ^ Cannot assign `undefined` to `x` because undefined [1] is incompatible with number [2].
```

TypeScript:

```ts
let x:number = undefined;
Type 'undefined' is not assignable to type 'number'.
```

also, it checks if the variable initialized or not

Flow:

```ts
let x:number;
x * 1;
^ Cannot perform arithmetic operation because uninitialized variable [1] is not a number.
```

TypeScript:

```
let x:number;
x * 1;
Variable 'x' is used before being assigned.
```

but be careful with Flow:

```ts
let x:number;
x + 1;
No errors!
```

TypeScript

```ts
let x:number;
x + 1;
Variable 'x' is used before being assigned.
```

### Option type or how to represent the absence of value with types

The good way to represent `null` value is so called tagged* unions - imagine you have a collection of values of different type, if you will attach some tag to each value by each you can clearly differentiate one value from another you can safely mix it in one bag. So you mark all actual values with one tag and you have one special value with a tag which represents a null value.

You can imagine it like this (rough example, you don't do it like this in JS):

```js
const taggedValues = [
  {
    type: "Some",
    value: 1
  },
  {
    type: "None"
  },
]
```

in ML languages you do not need to construct objects, you can make it with the help of types

```ocaml
type 'a option = None | Some of 'a
```

it can be roughly translated to Flow as

```ts
type None = void
type Some<a> = a
type Option<a> = None | Some<a>
// or simpler
type Option<a> = void | a
// even simpler - syntax sugar
type Option<a> = ?a
```

The native solution in Flow is called [Maybe type](https://flow.org/en/docs/types/maybe/).

TypeScript:

```ts
type Option<a> = void | a
```

If you use Option type, the system will make sure you do not apply any operation to the value unless you checked that the value is actually present.

```ts
let x:?number = 1;
x * 1;
^ Cannot perform arithmetic operation because null or undefined [1] is not a number.

let x:?number;
if (x != undefined) x * 1;
No errors!
```

TypeScript

```ts
let x:?number = 1;
x * 1;
No errors
```

but:

```ts
const t = (x:number|void) => x * 1;
The left-hand side of an arithmetic operation must be of type 'any', 'number' or an enum type.
```

Option type - is a safer alternative for a null value. (There is one more alternative - Maybe monad.)

### Verbosity of the Option type

Before you will be able to use Option type value in any operation (that requires exact type) you need to prove that the value is actually there:

```ts
let x:?number;
// some code which touches x

// x can be number or null or undefined at this point of code
if (x != undefined) {
  // x only can be a number at this point of code,
  // otherwise we will not get into this branch.
  // We can say that if condition proves that inside of this scope x is number.
  x * 1;
}
```

this approach can be verbose, that is why I try to use Option type only if it is required. For example, to describe the state of the form:

```ts
type State = {
  name?: string,
  age?: number
};
// initial state of the form
let state: State = {};
// user filled in first field
state = { name: 'Abc' };
// user filled in second field
state = { name: 'Abc', age: 20 };
```

but as soon as the user submits (and all fields required to be filled in, before the user can submit), we can use stricter types:

```ts
type User = {
  name: string,
  age: number
};
const onSubmit = (user: User) => { /* code */ };
```

This helps to fight with verbosity

## Word of caution

I advertised Option type so much, but there is one caveat in Option type impelemntation in Flow and TypeScript:

```ts
const a: Array<{c:1}> = [];
const b = a[0]
const d = b.c;
No errors!
const e: {[key: string]: {c:1}} = {};
const f = e.f;
f.c;
No errors!
```

It will not catch errors here! Keep this in mind.
