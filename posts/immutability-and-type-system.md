This is wrong article, it needs to be fixed

# Immutability and type system

Immutability can be useful. First of all, it is easier to work with immutable objects when you to share them between concurrent or parallel processes. Second, if you agree to use immutable objects it is much easier to do a comparison, it is enough to compare pointers.

## Easier to reason about concurrent processes

There are no real parallel processes in JS (at least this paradigm is not widespread), because JS is built around event-loop paradigm, but you still have to deal with concurrency. The most obvious example of concurrency is React's Fiber engine - to not block the browser for a long time they batch render in small portions and periodically return execution to the browser, but this also means that some state can change in the middle of the React's process. That is why they require for the state to be immutable and you can only set new one via `setState`.

## Fast comparison

Imagine you have expensive (in sense of CPU time) operation, you can improve performance by memoizing (caching) results of the computation. To be able to memorize (safely) function you need it to be without side effects (otherwise you will have to deal with other problems, like bugs in the logic and complex cache invalidations rules). Also, you would need to be able to compare arguments of function fast, the fastest way to do it is by comparing pointers (address in memory), otherwise you will need to compare each value (or property) of given object to be sure they are the same and this is not always possible.

```js
const memoize = f => {
  const cache = new WeakMap();
  return x => {
    if (cache.has(x)) return cache.get(x);
    const res = f(x);
    cache.set(x, res);
    return res;
  };
};
const sum = a => a.reduce((b, c) => b + c, 0);
const memoizedSum = memoize(sum);
const a = [1, 2, 3, 4];
sum(a) === memoizedSum(a); // ok
a.push(5);
sum(a) === memoizedSum(a); // Error (15 != 10)
```

### Flow

```ts
type F<I, O> = (x: I) => O;
// WeakMap only supports complex objects as keys,
// but there is no way to express this in Flow
const memoize = <I, O>(f: F<I, O>): F<I, O> => {
  const cache = new WeakMap<I, O>();
  return (x: I): O => {
    // cache.has(x) doesn't work for Flow in this case, ignore this for now
    let res = cache.get(x);
    if (res !== undefined) return res;
    res = f(x);
    cache.set(x, res);
    return res;
  };
};
```

Let's force read-only parameters

```ts
const memoize = <I, O>(f: F<$ReadOnlyArray<I>, O>): F<$ReadOnlyArray<I>, O> => {
  const cache = new WeakMap<$ReadOnlyArray<I>, O>();
  return (x: $ReadOnlyArray<I>): O => {
    // cache.has(x) doesn't work for Flow in this case, ignore this for now
    let res = cache.get(x);
    if (res !== undefined) return res;
    res = f(x);
    cache.set(x, res);
    return res;
  }
}
const sum = (a: $ReadOnlyArray<number>) => a.reduce((b, c) => b + c, 0);
const memoizedSum = memoize(sum);
const a: $ReadOnlyArray<number> = [1, 2, 3, 4];
memoizedSum(a);
a.push(5);
    ^ Cannot call `a.push` because property `push` is missing in `$ReadOnlyArray` [1].
```

### Immutability vs structural subtyping

Yes, Flow catches the error in the case described above and in this:

```ts
const sum = (a: Array<number>) => a.reduce((b, c) => b + c, 0);
const memoizedSum = memoize(sum);
                                ^ Cannot call `memoize` with `sum` bound to `f` because array type [1] is incompatible with read-only array type [2] in the first argument.
```

But not in this:

```ts
const sum = (a: $ReadOnlyArray<number>) => a.reduce((b, c) => b + c, 0);
const a = [1, 2, 3, 4];
sum(a);
No errors!
```

I guess this is because [Flow uses structural subtyping for objects](https://flow.org/en/docs/lang/nominal-structural/#toc-objects-are-structurally-typed)

`$ReadOnlyArray` has all read operations, but doesn't have write operations, like `push`. While `Array` has all read operations and all write operations, so from type system point of view `Array` argument fits everywhere where `$ReadOnlyArray` is required. To overcome this limitation we need to use [nominal typing](https://michalzalecki.com/nominal-typing-in-typescript/) or force uniqueness of type with [opaque types](posts/opaque-types.md). We can do something like:

```ts
opaque type Secret = "secret";
export type ROArray<T> = { t: Secret } & $ReadOnlyArray<T>;
export const toROArray = <T>(val: $ReadOnlyArray<T> | Array<T>): ROArray<T> => {
  //$FlowFixMe
  return [...val];
};
```

Remember that we need to keep it in separate files for `opaque` type to work as expected.

```ts
import { type ROArray, toROArray } from "./ROArray";
type F<I, O> = (x: I) => O;
const memoize = <I, O>(f: F<ROArray<I>, O>): F<ROArray<I>, O> => { /*...*/ }
```

Let's compare:

```ts
const mutable = [1, 2, 3, 4];
const immutable: $ReadOnlyArray<number> = [...mutable];
const immutableOpaque = toROArray(b);
const withMutable = (x: Array<number>) => null;
const withImmutable = (x: $ReadOnlyArray<number>) => null;
const withImmutableOpaque = (x: ROArray<number>) => null;
```

|                       | `mutable` | `immutable` | `immutableOpaque` |
| --------------------- | --------- | ----------- | ----------------- |
| `withMutable`         | ok        | - (2)       | - (4)             |
| `withImmutable`       | ok        | ok          | ok                |
| `withImmutableOpaque` | - (1)     | - (3)       | ok                |

1. `Cannot call withImmutableOpaque with mutable bound to x because array literal [1] is incompatible with object type [2].`
2. `Cannot call withMutable with immutable bound to x because read-only array type [1] is incompatible with array type [2].`
3. `Cannot call withImmutableOpaque with immutable bound to x because read-only array type [1] is incompatible with object type [2].`
4. `Cannot call withMutable with immutableOpaque bound to x because read-only array type [1] is incompatible with array type [2].`

### TypeScritpt

```ts
type F<I, O> = (x: I) => O;
// `WeakMap` only supports complex objects as keys,
// this is why we use `I extends Object`
const memoize = <I extends Object, O>(f: F<I, O>): F<I, O> => {
  const cache = new WeakMap<I, O>();
  return (x: I): O => {
    // cache.has(x) doesn't work for TypeScript in this case, ignore this for now
    let res = cache.get(x);
    if (res !== undefined) return res;
    res = f(x);
    cache.set(x, res);
    return res;
  }
}
```

Let's force read-only parameters

```ts
type F<I, O> = (x: I) => O;
// `WeakMap` only supports complex objects as keys,
// this is why we use `I extends Object`
const memoize = <I, O>(f: F<ReadonlyArray<I>, O>): F<ReadonlyArray<I>, O> => { /*...*/ }
const sum = (a: ReadonlyArray<number>) => a.reduce((b, c) => b + c, 0);
const memoizedSum = memoize(sum);
const a: ReadonlyArray<number> = [1,2,3,4];
memoizedSum(a);
a.push(5);
Property 'push' does not exist on type 'ReadonlyArray<number>'.
```

The same issue as in Flow case and structural subtyping:

```ts
const a: Array<number> = [1,2,3,4];
memoizedSum(a);
No errors!
```

**TODO**: add similar to Flow's section about structural subtyping and immutability

## Other things worth to mention

### Immutable operations in JS

As soon as you start work with immutable data you will notice, that you can not mutate things, instead, you forced to create a new version of data each time. To create a new version of data you will need to disassemble one immutable structure and assemble a new one (modified). With ES6 spread operator, it is a bit easier

```js
const newObject = {
  ...oldObject,
  newValue
};
const newArray = [...newArray, newValue];
```

See the list of immutable operations for arrays in [this article](https://vincent.billey.me/pure-javascript-immutable-array/).

Some disadvantages of this (naive) approach: you will have a lot of copies of similar data, creating those copies can be slow, it is very inconvenient to create modified versions of deeply nested objects.

### Functional aka persistent data structures

Instead of copying the whole immutable object you can create a new one which will contain only modifications (diff) and a pointer to the previous version, this is faster and takes less memory. This is the main idea behind [functional data structures](https://www.cs.cmu.edu/~rwh/theses/okasaki.pdf).

Here are some examples of functional data structures from JS world: [immutable-js](https://facebook.github.io/immutable-js/), [mori](https://github.com/swannodette/mori).

### Local mutability and global immutability

I picked up this idea from [the talk by Julien Verlaguet "Reflex: Reactive Programming at Facebook"](https://youtu.be/AGkSHE15BSs). Immutability required only when variable leaves the scope of the function where it is allowed to mutate, like in reducer in `Redux` or `setState` in `React`. The main purpose is to make sure that the change of state is controlled.

[immer](https://github.com/mweststrate/immer/) explores this idea in JS world. I haven't tried it yet, so can't say much about it.

### Lenses

I will not get into details in this post explaining what lenses are, but it is required to mention, that lenses (and prisms aka profunctor optics) can be used to simplify updates of deeply nested structures, including immutable structures.

### Referential transparency and immutability

I need mention that referential transperancy is not the same as immutability, because they often beeing confused. The easiest way to explain is by example.

**Referential transperancy**:

```ts
const a = { x: 0 };
a.x = 1;
a = { x: 1 };
```

Flow: `^ Cannot reassign constant 'a' [1].`

TypeScript: `Cannot assign to 'a' because it is a constant or a read-only property.`

**Immutability**:

Flow:

```ts
let a: $ReadOnly<{x: number}> = { x: 0 };
a.x = 1;
  ^ Cannot assign `1` to `a.x` because property `x` is not writable.
a = { x: 1 };
```

TypeScript:

```ts
let a: Readonly<{x: number}> = { x: 0 };
a.x = 1;
  Cannot assign to 'x' because it is a constant or a read-only property.
a = { x: 1 };
```
