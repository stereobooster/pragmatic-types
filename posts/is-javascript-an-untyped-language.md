# Is JavaScript an untyped language?

> I've found that some people call JavaScript a "dynamically, weakly typed" language, but some even say "untyped"? Which is it really?
>
> -- https://stackoverflow.com/questions/964910/is-javascript-an-untyped-language

To answer the question we need to define what is "untyped", what is "dynamically" and "weakly" typed languages are - read the full post on the subject ["Dynamically-, statically-, gradually-, weakly-, strongly- and un-typed languages"](posts/dynamic-static-gradual-untyped.md). If you decided to skip it:

 - Untyped - language with one type, like assembly language which works with the only one type - bit strings.
 - Dynamically typed or better to say dynamically-checked types - language in which types get checked at runtime.
 - Weakly typed - this term doesn't have an exact meaning, so I advise to avoid it, but most likely people refer to JavaScript's implicit coercions, which make types look "weak".

## JS and types
There are seven possible values that `typeof` returns: "number," "string," "boolean," "object," "function," "undefined," and "unknown". Also, we can check if values are an instance of some type like this:

```js
date instanceof Date
```

or like this

```js
Object.prototype.toString.call(date) === '[object Date]'
```

No, **JS is not untyped**. It has more than one type.

## Is JavaScript a dynamically typed language?

Type checking "performed by system" at runtime:
```js
undefined()
VM308:1 Uncaught TypeError: undefined is not a function
    at <anonymous>:1:1
```

Type checking "performed by programmer" at runtime:
```js
if (typeof x === "string")
```

Yes, **JS is dynamically typed**.

## Why people get so confused about this subject?
JS goes an extra mile to pretend it doesn't have types or type errors.

### What type errors exist in JS?

When you try to use non-function value as a function
```js
undefined()
VM308:1 Uncaught TypeError: undefined is not a function
    at <anonymous>:1:1
```

When you try to access a property of `undefined` or `null` value. 
Other values considered to be an object and if you access an inexisting value of an object you will get `undefined` instead of type error. This is hidden type error.
```js
null.test
VM84:1 Uncaught TypeError: Cannot read property 'test' of null
    at <anonymous>:1:1
undefined.test
VM134:1 Uncaught TypeError: Cannot read property 'test' of undefined
    at <anonymous>:1:1
```

Arithmetic operations on non-number values result in `NaN`, which is the JS way to express TypeErrors about arithmetic operations
```
1 * {}
NaN
```

### Coercion
Coercion can be convenient when you do not need explicitly convert from integers to floating point numbers in arithmetic operations, but in JS coercion used to hide type errors.
```
1*"1" // result 1, should be type error
1+"1" // result "11", should be type error
1*[] // result 0, should be type error
1+[] // result "1", should be type error
"1"+[] // result "1", should be type error
```

JS tries to hide type errors so hard, that it resulted in obscure coercion rules: https://twitter.com/angealbertini/status/979254093846859777

There a lot of research on that matter:
- [Wat](https://www.destroyallsoftware.com/talks/wat), A lightning talk by Gary Bernhardt from CodeMash 2012. About coercions in JS
- [WTFJS](https://www.youtube.com/watch?v=et8xNAc2ic8), 2012; [github repo](https://github.com/denysdovhan/wtfjs)
- [What the... JavaScript?](https://www.youtube.com/watch?v=2pL28CcEijU), 2015; [github repo](https://github.com/getify/You-Dont-Know-JS)

### Inconsistent types
JS have bugs in type operators which are preserved until now for compatibility reasons. For example:
```js
typeof null
"object"
```

Bugs in type operators and obscure coercion rules make an impression that there is no way such language have types.

## Final conclusion

JS has types, JS has type errors even so it tries to hide most of them, JS can check types at runtime. JS is definitely dynamically typed language.
