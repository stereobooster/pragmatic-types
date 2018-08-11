# Opaque types

> The 'root cause' of the loss of the spacecraft was the failed translation of English units into metric units in a segment of ground-based, navigation-related mission software, as NASA has previously announced.
> -- [source](https://mars.jpl.nasa.gov/msp98/news/mco991110.html)

It sounds almost unrealistic - a software bug led to spacecraft loss. But this is true developer forgot to translate one type of units into another type of units. 

How to make sure you will not add meters to miles or meters to seconds or seconds to hours or Euros to Dollars? Type systems have an answer for it - opaque types.

## Flow

Imperial.js:
```ts
// @flow
export opaque type Mile = number;
export const numberToMile = (n: number): Mile => n;
```

Metric.js:
```ts
// @flow
export opaque type Kilometer = number;
export const numberToKilometers = (n: number): Kilometer => n;
```

test.sj
```ts
//@flow
import { type Kilometer } from './Metric'
import { numberToMile } from './Imperial'
export const calculateOrbit = (n: Kilometers) => {
  // do some math here
  n;
};

let m = numberToMile(123);
calculateOrbit(m);
```

Error:
```
Cannot call calculateOrbit with m bound to n because Mile [1] is incompatible with Kilometers [2].

     test.js
 [2]  4│ export const calculateOrbit = (n: Kilometer) => {
      5│   // do some math here
      6│   n;
      7│ };
      8│
      9│ let m = numberToMile(123);
     10│ calculateOrbit(m);
     11│
```

**Note**: this will work only if you keep definitions of `Mile` and `Kilometer` in separate files.

## TypeScript

There is no native opaque type in TypeScript, but you can use [a solution proposed by Charles Pick](https://codemix.com/opaque-types-in-javascript/):

```ts
type Opaque<K, T> = T & { __TYPE__: K };
```

[Example](https://www.typescriptlang.org/play/#src=type%20Opaque%3CK%2C%20T%3E%20%3D%20T%20%26%20%7B%20__TYPE__%3A%20K%20%7D%3B%0D%0A%0D%0Atype%20Kilometer%20%3D%20Opaque%3C'Kilometers'%2C%20number%3E%3B%0D%0Atype%20Mile%20%3D%20Opaque%3C'Mile'%2C%20number%3E%3B%0D%0Aconst%20numberToMile%20%3D%20(n%3A%20number)%20%3D%3E%20n%20as%20Mile%3B%0D%0Aconst%20calculateOrbit%20%3D%20(n%3A%20Kilometer)%20%3D%3E%20%7B%0D%0A%20%20%2F%2F%20do%20some%20math%20here%0D%0A%20%20n%3B%0D%0A%7D%3B%0D%0A%0D%0Alet%20m%20%3D%20numberToMile(123)%3B%0D%0AcalculateOrbit(m)%3B%0D%0A):
```ts
type Kilometer = Opaque<'Kilometers', number>;
type Mile = Opaque<'Mile', number>;
const numberToMile = (n: number) => n as Mile;
const calculateOrbit = (n: Kilometer) => {
  // do some math here
  n;
};

let m = numberToMile(123);
calculateOrbit(m);
```

Error:
```
Argument of type 'Opaque<"Mile", number>' is not assignable to parameter of type 'Opaque<"Kilometers", number>'.
  Type 'Opaque<"Mile", number>' is not assignable to type '{ __TYPE__: "Kilometers"; }'.
    Types of property '__TYPE__' are incompatible.
      Type '"Mile"' is not assignable to type '"Kilometers"'.
```
