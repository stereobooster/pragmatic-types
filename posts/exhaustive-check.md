# Exhaustive check

It is useful in event processing, or anywhere where you have to deal with pattern matching (like `switсh/case` in JS) and disjoint unions (aka unions or enums). 

Example: assume we have events (or actions in Redux) and we have a function which supposes to process events (or reducer in Redux)

```js
const addTodo = {
  type: 'ADD_TODO',
  payload: 'buy juice'
}
const removeTodo = {
  type: 'REMOVE_TODO',
  payload: 'buy juice'
}
function process(event) {
  switch(event.type) {
    case 'ADD_TODO': // do something
      break;
    case 'REMOVE_TODO': // do something
      break;    
  }
}
```

Next task is to add one more type of event:

```js
const addTodo = {
  type: 'CHANGE_TODO',
  payload: {
    old: 'buy juice',
    new: 'buy water',
  }
}
```

We need to keep in mind all the places in the code where we need to change behavior. This is easy if it is two consequent tasks and if there is only one place to change. But what if we need to do it two months later, and you didn't write code in the first place. This sounds harder right. And if this system is in production and change in critical part you will have FUD (Fear-Uncertainty-Doubt).

This is where an exhaustive check shines in.

## Flow

```ts
function exhaustiveCheck(value: empty) {
  throw new Error(`Unhandled value: ${value}`)
}

type AddTodo = {
  type: 'ADD_TODO',
  payload: string
}
type RemoveTodo = {
  type: 'REMOVE_TODO',
  payload: string
}
type Events = AddTodo | RemoveTodo;

function process(event: Events) {
  switch(event.type) {
    case 'ADD_TODO': // do something
      break;
    case 'REMOVE_TODO': // do something
      break;
    default:
      exhaustiveCheck(event.type);
  }
}
```

As soon as you add a new event (new value to a union type)  type system will notice it and complain.

```ts
type ChangeTodo = {
  type: 'CHANGE_TODO',
  payload: string
}
type Events = AddTodo | RemoveTodo | ChangeTodo;
```

Will result in ([try yourself](https://flow.org/try/#0GYVwdgxgLglg9mABAUwB4AsCGIDOsBuyAwushANYAU+mANiMgFwoC2ADlAJ4CUiA3gChEiKOgBOcAO6IwyaQFExEsZQAGAVTBYwAE1rIdiGvSaIAJH2MMAvqu4DrAgVzbJEAQR06AKnB1xEAF5+IRFOV2YAcncAERiAfW8AeRikyIAaULZMTlo4TB1mPDEYMABzB2dwtwAlZBY4Ql9-IJDhF1NImvkAWSSANXlElLTM4Wzc-MLEYtKKxw7EEkxy5GaA4MF26qiiAAl3ADkAcSHk1IysnLyCoqgS8srF+UIwKBxWzx8-AIAfRDqDSaP0Q-2Wq3WAG4nKBILAEIg2BIIMgcDhKMhXlBmC9kG8cLwtjNJDAoBB0BisQA6DqE0LCCCYHBuaJxYYXZgAek5iBaODgLGQojm9OEiAARmJkJhyNCxYhGczEF1egMziNIlyeXyBUL0CL5cJJdLZaKdMhgNhaNjRcI0FhcARiKQKJS8VAadVuHLEI5rEA)):

```
26:       exhaustiveCheck(event.type);
                          ^ Cannot call `exhaustiveCheck` with `event.type` bound to `value` because string literal `CHANGE_TODO` [1] is incompatible with empty [2].
References:
14:   type: 'CHANGE_TODO',
            ^ [1]
1: function exhaustiveCheck(value: empty) {
                                   ^ [2]
```

## TypeScript

[TypeScript example](https://www.typescriptlang.org/play/index.html#src=function%20exhaustiveCheck(value%3A%20never)%20%7B%0D%0A%20%20throw%20new%20Error(%60Unhandled%20value%3A%20%24%7Bvalue%7D%60)%0D%0A%7D%0D%0A%0D%0Atype%20AddTodo%20%3D%20%7B%0D%0A%20%20type%3A%20'ADD_TODO'%2C%0D%0A%20%20payload%3A%20string%0D%0A%7D%0D%0Atype%20RemoveTodo%20%3D%20%7B%0D%0A%20%20type%3A%20'REMOVE_TODO'%2C%0D%0A%20%20payload%3A%20string%0D%0A%7D%0D%0Atype%20ChangeTodo%20%3D%20%7B%0D%0A%20%20type%3A%20'CHANGE_TODO'%2C%0D%0A%20%20payload%3A%20string%0D%0A%7D%0D%0Atype%20Events%20%3D%20AddTodo%20%7C%20RemoveTodo%20%7C%20ChangeTodo%3B%0D%0A%0D%0Afunction%20process(event%3A%20Events)%20%7B%0D%0A%20%20switch(event.type)%20%7B%0D%0A%20%20%20%20case%20'ADD_TODO'%3A%20%2F%2F%20do%20something%0D%0A%20%20%20%20%20%20break%3B%0D%0A%20%20%20%20case%20'REMOVE_TODO'%3A%20%2F%2F%20do%20something%0D%0A%20%20%20%20%20%20break%3B%0D%0A%20%20%20%20default%3A%0D%0A%20%20%20%20%20%20exhaustiveCheck(event.type)%3B%0D%0A%20%20%7D%0D%0A%7D) looks the same except instead of `empty` use `never`:

```ts
function exhaustiveCheck(value: never) {
  throw new Error(`Unhandled value: ${value}`)
}
```

The error looks like:

```
Argument of type '"CHANGE_TODO"' is not assignable to parameter of type 'never'.
```
