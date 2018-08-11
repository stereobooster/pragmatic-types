# IO Validation

Languages with static types need a special procedure to convert data from the outside (untyped) world (aka Input-Output or IO) to internal (typed) world. Otherwise, they will lose promised type safety. This procedure is called IO validation. Side note: the fact that system makes type checking at run-time means it is a dynamically-typed system, but this will be explained in another post.

A typical example of IO validation is parsing of JSON response from API.

## Flow and TypeScript
Note: code looks identical in TypeScript and Flow

```ts
// @flow
type Person = {
  name: string;
};
// $FlowFixMe or @ts-ignore
const getPerson = (id: number): Promise<Person> =>
  fetch(`/persons/${id}`).then(x => x.json());
```

We want that `getPerson` would return `Promise` of `Person`, and we tricked type system to believe that it always be the case, but in reality, it can be anything. What if API response look like:

```js
{
  "data": { "name": "Jane" },
  "meta": []
}
```

This would end up being runtime error somewhere in function which expects `Person` type. So even our static type system doesn't find errors they still potentially exist. Let's fix this by adding IO validation.

```ts
// it is guaranteed that this function will return a string
const isString = (x: any): string => {
  if (typeof x !== "string") throw new TypeError("not a string");
  return x;
};

// it is guaranteed that this function will return an object
const isObject = (x: any): { [key: string]: any } => {
  if (typeof x !== "object" || x === null) throw new TypeError("not an object");
  return x;
};

// it is guaranteed that this function will return an Person-type
const isPerson = (x: any): Person => {
  return {
    name: isString(isObject(x).name)
  };
};
```

Now we have a function which will guaranteed return Person or throw an error, so we can do:

```ts
// without need to use $FlowFixMe
const getPerson = (id: number): Promise<Person> =>
  fetch(`/persons/${id}`)
    .then(x => x.json())
    .then(x => {
      try {
        return isPerson(x);
      } catch (e) {
        return Promise.reject(e);
      }
    });
```

or if we take into account that any exception thrown inside Promise will turn into rejected promise we can write:

```ts
// without need to use $FlowFixMe
const getPerson = (id: number): Promise<Person> =>
  fetch(`/persons/${id}`)
    .then(x => x.json())
    .then(x => isPerson(x));
```

This is the basic idea behind building a bridge between dynamic and static type systems. A full example in Flow is [here](https://flow.org/try/#0CYUwxgNghgTiAEAzArgOzAFwJYHtVJAzAAsAKAZwxi1QHMAaeHAB2z3IH4AueAbwF8AlDwAKMHAFss5EAB5eAK3J4upQQF4AfGMnS5UVAE9N-TQG4AUAHor8AAKIIOAO4WMh5ghEgYy-Or4LeHhUKAkQHkpqOkt+SzB2DHhpAGUqGlp4ANIADx4DQ2F4KIyszUDgrER4UndPHGqc+ABCdQCAIhK6dsF4DGJxZxCQIYAVDxAAURhxGFJ21BwkqGL07sFLYLgMZBh8HNj4xOTyAHkAIwVwJOy8+AKi3ngAbQBrEENItdoAXXyjeD8MoVZLVWoTBrwJqtDo4S7XdrwAA+SKhWTaIWQEAgvX6g2GYwm01m80Wy3wcKumB6m3g212+0OFgSqEoJ28vjwWRqdweoh8fmBvCCdMIDJBwVC4R4qW+pGkFypGFyggAdFKQIIRXELHFgszjrRCBzBdksMAeKhkBJzj4ijopDJZCa8OUtCLEIQSKQAAZWTyc1lWAAkvHN-B9WuCwVV-RAqFywJyqqUeDUUejseI8cTWnZArTOUEGyAA). A full example in TypeScript is [here](https://www.typescriptlang.org/play/index.html#src=type%20Person%20%3D%20%7B%0D%0A%20%20name%3A%20string%3B%0D%0A%7D%3B%0D%0Aconst%20isString%20%3D%20(x%3A%20any)%3A%20string%20%3D%3E%20%7B%0D%0A%20%20if%20(typeof%20x%20!%3D%3D%20%22string%22)%20throw%20new%20TypeError(%22not%20a%20string%22)%3B%0D%0A%20%20return%20x%3B%0D%0A%7D%3B%0D%0Aconst%20isObject%20%3D%20(x%3A%20any)%3A%20%7B%20%5Bkey%3A%20string%5D%3A%20any%20%7D%20%3D%3E%20%7B%0D%0A%20%20if%20(typeof%20x%20!%3D%3D%20%22object%22%20%7C%7C%20x%20%3D%3D%3D%20null)%20throw%20new%20TypeError(%22not%20an%20object%22)%3B%0D%0A%20%20return%20x%3B%0D%0A%7D%3B%0D%0Aconst%20isPerson%20%3D%20(x%3A%20any)%3A%20Person%20%3D%3E%20%7B%0D%0A%20%20return%20%7B%0D%0A%20%20%20%20name%3A%20isString(isObject(x).name)%0D%0A%20%20%7D%3B%0D%0A%7D%3B%20%20%0D%0Aconst%20getPerson%20%3D%20(id%3A%20number)%3A%20Promise%3CPerson%3E%20%3D%3E%0D%0A%20%20fetch(%60%2Fpersons%2F%24%7Bid%7D%60)%0D%0A%20%20%20%20.then(x%20%3D%3E%20x.json())%0D%0A%20%20%20%20.then(x%20%3D%3E%20isPerson(x))%3B)

## Libraries
It is not very convenient to write those kinds of validations every time by hand, instead, we can use some library to do it for us. 

### [sarcastic](https://github.com/jamiebuilds/sarcastic/) for Flow
Minimal, possible to read the source and understand. **Cons**: [misses the `union` type](https://github.com/jamiebuilds/sarcastic/pull/3).

```ts
import is, { type AssertionType } from "sarcastic"
const PersonInterface = is.shape({
  name: is.string
});
type Person = AssertionType<typeof PersonInterface>
const assertPerson = (val: mixed): Person =>
  is(val, PersonInterface, "Person")
const getPerson = (id: number): Promise<Person> =>
  fetch(`/persons/${id}`)
    .then(x => x.json())
    .then(x => assertPerson(x));
```

### [io-ts](https://github.com/gcanti/io-ts) for TypeScript
Good, advanced, with FP in the heart.

```ts
import * as t from "io-ts"
const PersonInterface = t.type({
  name: t.string
});
type Person = t.TypeOf<typeof Person>
const getPerson = (id: number): Promise<Person> =>
  fetch(`/persons/${id}`)
    .then(x => x.json())
    .then(x => PersonInterface.decode(x).fold(
       l => Promise.reject(l),
       r => Promise.resolve(r)
     ));
```

## Generator
No need to write "IO validators" by hand, instead [we can use tool to generate it from JSON response](https://transform.now.sh/). Also, check [type-o-rama](https://github.com/stereobooster/type-o-rama) for all kind of conversion of types. Generators with IO validation marked by box emoji.
