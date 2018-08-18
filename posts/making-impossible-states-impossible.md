# Making Impossible States Impossible

> Making Impossible States Impossible
> -- by [Richard Feldman](https://www.youtube.com/watch?v=IcgmSRJHu_8), Elm

> Making Unreasonable States Impossible
> -- by [Patrick Stapfer](https://www.youtube.com/watch?v=P7dTPoxCg4w), ReasonML

![](making-impossible-states-impossible/penrose-triangle.png?raw=true)

[source](https://en.wikipedia.org/wiki/Penrose_triangle#/media/File:Penrose-dreieck.svg)

## Impossible states

Impossible states, or nonsense states - states of the system, which doesn't make any sense, most likely they are a by-product of how you store your state. For example, you are doing AJAx request:

- you can store loading state `{ loading: true }`
- you can store if response successful or errored `{ ok: true }`

Two boolean variables give you four combainations:

|             | `loading: false` | `loading: true` |
|-------------|------------------|-----------------|
| `ok: false` | 1                | 2               |
| `ok: true`  | 3                | 4               |

1 and 3 make sense, but 2 and 4 not really. This is a typical problem when boolean variable get used instead of `enum` (or `union`)

```ts
type RequestState = 'loading' | 'ok' | 'error'
```

**Example 2**: two selects, first one country, second one city. The state can be described as:

```ts
type Location = {
  country: string | null,
  city:    string | null
};
```

As soon as the user selects a country, system updates list of cities. If the user selects a country, selects a city, selects empty value as a country at this point we need to set the city to empty value too, but our types permit any combination of states. Let's change this:

```ts
type Location = $ReadOnly<
  | {| country: null,   city: null   |} // on country de-select
  | {| country: string, city: null   |} // on country select or on city de-select
  | {| country: string, city: string |} // on city select
>;
```

See [Richard Feldman's talk](https://www.youtube.com/watch?v=IcgmSRJHu_8) for other examples.

## What types have to do with impossible states?

Type system helps you to prevent bugs before they will get to production, but it can not catch all bugs unless you help it. If you will make sure your types don't permit invalid states your type system will take care to make sure your program will not get into the wrong state.

### Not a panacea

"Making Impossible States Impossible" approach will not help if there is no way to express impossibility via type system. For example, we have a state for pie chart, each entry in a state represents the section in a chart and has name and value(%), for the state to be valid we need that sum of all values equals to 100%. In type systems like Flow or TypeScript (or Reason or PureScript) there is no way to restrict the impossible state from type system point of view. To handle this kind of situation you need a more advanced type system with dependent types, like Agda. This problem is similar to the problem of division by zero - from type system point of view 0 can be used as divisor because it is number, restrict this we need type system which can say this function accepts "all numbers except 0".

Side note: it is possible to solve the problem with pie chart by using absolute values instead of percents and calculate percents on the fly, but this is just an example.

"Making the Impossible States Impossible" is just one of the type of so-called business logic errors which is possible to prevent with the help of the type system. But there are others, for example, impossible transitions. When business rules require a specific transition from one state to another. That kind of requirements is not (easily) expressable with the help of the described technique. This task is typically solved with Finite State Machines. I tried to come up with a good example for a system which doesn't allow impossible states, but still can have bugs with the impossible transition and wasn't able to do it right away, so **take this paragraph with a grain of salt**, maybe it is not the whole truth.

"Making the Impossible States Impossible" doesn't prevent infinite loops and doesn't prove that all states are reachable. Keep in mind, that this technique will greatly decrease the number of bugs in your application, but this doesn't mean that you [formally proofed correctness of it](https://www.hillelwayne.com/post/theorem-prover-showdown/).
