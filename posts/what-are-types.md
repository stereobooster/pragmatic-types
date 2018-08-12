# What are types?
The idea of this post is to give you a framework to reason about types (in programming), I'm not gonna try to give an exhaustive and full mathematically correct definition of types.

[Also, some mathematicians argue there is no single definition of the types and this is good.](http://tomasp.net/academic/papers/against-types/)

## Definition
Type is a collection of items, often having some common properties, structure and operations permitted for this type.

For example 

```
Cars: 🚙, 🚌, 🚜. 
Fruits: 🍋, 🍐, 🍓.
```

Also, we can define operations for those types

```
To drive <a car>
To eat <a fruit>
```

So we can do this

```
To drive 🚙
To eat 🍋
```

But what happens if we confuse things

```
To drive 🍋
```

What does it mean to drive fruit? What is the result of this operation?

```
TypeError: you can not drive fruits
```

The result is nonsense. Also, you can say that this is an undefined value in the same way as the result of division by zero is undefined in math. Humankind doesn't know the answer.

As you can see types and type errors are not computer specific thing. Types exist because of humans. Humans like to discover patterns, group objects by properties and later make conclusions about the whole group.

Now let's see if our ideas about types hold true in the computer world.

```
Cars: 🚙, 🚌, 🚜.     → Number
Fruits: 🍋, 🍐, 🍓.   → String

To drive 🚙            → To multiply numbers
To eat 🍋              → To concatenate strings
```

[Flow](https://flow.org/try/#0EQQ2AICpwRiA)

```
"a" * 1
Cannot perform arithmetic operation because string [1] is not a number.
```

[TypeScript](http://www.typescriptlang.org/play/#src=%22a%22%20*%201)

```
"a" * 1
The left-hand side of an arithmetic operation must be of type 'any', 'number' or an enum type.
````

[Reason](https://reasonml.github.io/en/try.html?reason=EQQ2AICpwRiA)

````
"a" * 1
Line 1, 8: This expression has type string but an expression was expected of type int
````

JavaScript

```
"a" * 1
NaN
```

NaN stands for not a number. [This is how IEEE (Institute of Electrical and Electronics Engineers) calls nonsense values for arithmetic operations.](https://stackoverflow.com/questions/14682005/why-does-division-by-zero-in-ieee754-standard-results-in-infinite-value)

## NaN and how to handle errors
There are two ways to handle errors from a machine point of view:

1. Raise exception. 

CPU will stop the execution of current instructions and jump to error handling function

2. Return special value which represents an error

CPU will continue to execute current instructions

Those special nonsense values are tricky because you can not do anything with it

```
nonsense + 1 = ? (nonsense)
nonsense * 1 = ? (nonsense)
nonsense / 1 = ? (nonsense)
```

As soon you get one value somewhere in the middle of a calculation, it will manifest to the end of the calculation. This is also called toxic value 💀. Once it gets into the system everything is poisoned.

Those values hard to debug, because the result of the error can be found far away from the place where the error happened and there are no traces left. This is why it is highly discouraged to use it.

## What is type checking? 
The answer is trivial - this is when types get checked, this is when you check that given thing is a member of the collection or not, to prevent errors as described above.

### Type checking "performed by a system"
```
undefined()
VM180:1 Uncaught TypeError: undefined is not a function
    at <anonymous>:1:1
```

### Type checking "performed by a developer"
```js
if (typeof x === "undefined") {}
```

### Dynamic type checking or type checking at runtime
```
undefined()
VM180:1 Uncaught TypeError: undefined is not a function
    at <anonymous>:1:1
```

### Static type checking or type checking before runtime
```ts
// @flow
undefined()
   ^ Cannot call `undefined` because undefined [1] is not a function.
```
