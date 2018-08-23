# Pragmatic types

[![Tweet][twitter-badge]][twitter]

## Introduction

### What
Small practical guide on statically-typed languages for developers who switching from dynamically-typed languages to statically-typed versions. For example from JavaScript to TypeScript or Flow.

### For who
This guide assumes at least some prior knowledge of dynamically-typed languages. The idea of this guide is to help make a mental shift from dynamic to static types.

Most of the time I will use JavaScript and TypeScript or Flow examples. Maybe this will be useful for other languages - I do not know yet, let's see.

If you have prior knowledge of statically typed languages, but Hindley-Milner doesn't ring a bell for you, this guide can be useful for you too.

### Why
This is my diary of personal discoveries. I find myself explaining those things again and again to different people. I decided to write it down once and for all.

## Table of contents
- [Exhaustive check and why it is useful for event processing, enums, unions or disjoint unions](posts/exhaustive-check.md)
- [Opaque types and how they could have saved Mars Climate Orbiter](posts/opaque-types.md)
- [IO validation or how to handle JSON-based APIs in statically typed language](posts/io-validation.md)
- [What are types?](posts/what-are-types.md)
- [Dynamically-, statically-, gradually-, weakly-, strongly- and un-typed languages](posts/dynamic-static-gradual-untyped.md)
- [Is JavaScript an untyped language?](posts/is-javascript-an-untyped-language.md)
- [Type system vs null](posts/type-system-vs-null.md)
- [Making Impossible States Impossible](posts/making-impossible-states-impossible.md)
- [Immutability and type system](posts/immutability-and-type-system.md)
- Classes, types, sets, structural and nominal subtyping, Liskov substitution principle
- Redux and impossible states
- Redux and impossible state transitions or FSM
- Redux and side effects
- Types and proofs
- Why JavaScript Promise is unsound?
- Terminology: soundness, completeness, totality, decidability
- Rosetta stone, Curry–Howard correspondence
- To be continued...

## Similar works

- [To type or not to type: quantifying detectable bugs in JavaScript](https://blog.acolyer.org/2017/09/19/to-type-or-not-to-type-quantifying-detectable-bugs-in-javascript/) Gao et al., ICSE 2017
- [Type systems will make you a better JavaScript programmer](https://jaredforsyth.com/type-systems-js-dev/#/) by Jared Forsyth 2017
- [Types, and Why You Should Care](https://www.youtube.com/watch?time_continue=1&v=0arFPIQatCU)
- [Lauren Tan Swipe Left, Uncaught TypeError Learning to Love Type Systems](https://youtu.be/y3uXazpAdwo)

## Contribution
Ideas, questions, and fixes are welcome. Sharing on social networks is also a big help. Thank you.

[twitter]: https://twitter.com/intent/tweet?text=Check%20out%20small%20practical%20guide%20on%20Flow%20and%20TypeScript%20for%20JavaScript%20developers%0A%20by%20%40stereobooster%20https%3A%2F%2Fgithub.com%2Fstereobooster%2Fpractical-types%20%F0%9F%91%8D
[twitter-badge]: https://img.shields.io/twitter/url/https/github.com/stereobooster/react-ideal-image.svg?style=social
