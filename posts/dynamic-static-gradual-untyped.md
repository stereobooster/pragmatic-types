# Dynamically-, statically-, gradually-, weakly-, strongly- and un-typed languages

Please read [the chapter "What are types?"](posts/what-are-types.md) before this one. In case you decided to skip the previous chapter here is what you need to know: I define **type** as a collection of things and **type checking** as the process of detecting if the thing belongs to the collection or not to prevent nonsense operations.

## Dynamically-typed or Dynamic types or Dynamic type system or Dynamically-checked types
If you change dynamically-typed to dynamically-checked types it becomes more clear on what this is about - it means that type checking happens **at runtime**. 

### Conceptual point of view
Conceptually any type checking at runtime can be considered as a dynamic type system, it includes IO validation, for example when a program reads a string from the input and check that this is number or not. 

### Implementation point of view
From an implementation point of view for language to be dynamically typed it needs to have data about type at runtime - every value stored in the memory should also store some flag or pointer to the type descriptor.

In OOP languages, like Ruby or Smalltalk, each value stores a pointer to the class because methods are stored in the classes and shared between all instances (there are some exclusions to this rule but let's not focus on this). Which makes some OOP languages automatically dynamically typed.

If you have the ability to introspect types at runtime or compare types at runtime or do pattern matching against types or reflection - all of those can be considered dynamic types from an implementation point of view.

I prefer to use a conceptual point of view because it is much easier to differentiate, you do not need to know the implementation details.

## Statically-typed or Static types or Static type system or Statically-checked types 
If you change statically-typed to statically-checked types it becomes more clear on what this is about - it means that type checking happens **before runtime**, it can be one step among others, like a compilation, transpiration, linting.

## Dynamic vs Static types
Most of the time dynamic types considered to be the opposite position to static types. There is no conflict in dynamic and static types, **languages can be same time dynamically-checked and statically-checked**. 

## Gradual type system
There is a trend to add static type analyzers to existing dynamic languages, for example, Flow for Javascript, Diamondback and Sorbet for Ruby. This kind of systems, which "gradually" introduce static type checking to dynamically-checked languages is what people call gradual type system.

## Untyped languages
 If you read the chapter "What are types?" you know that types are more a conceptual thing to prevent nonsense situations like applying an operation to an inappropriate type. How would you avoid this kind of situations to be able to refuse from types on a conceptual level? Easy - make sure there is only one type in the system, for example, assembly language - all values are bit strings.  When there is only one type there is no way you can get into trouble applying operation to the wrong type of value.

Untyped language - is the language with the only one type (or language without variables which would not have big area of application).

> The road from untyped to typed universes has been followed many times in many different fields, and largely for the same reasons. Consider, for example, the following untyped universes:
> (1) Bit strings in computer memory
> (2) S-expressions in pure Lisp
> (3) λ-expressions in the λ-calculus
> (4) Sets in set theory
>
> -- [On Understanding Types, Data Abstraction, and Polymorphism](http://lucacardelli.name/Papers/OnUnderstanding.A4.pdf)

## Weak and strong type system
There is absolutely no way to define what that terminology means, so avoid it to prevent confusion in conversations. It is possible to say that one type system is more powerful than another, for example, Haskell's type system is more powerful than Go's, and Agda's is more powerful than Haskel's (I guess), but standalone terminology "weak" and "strong" doesn't make much sense.

Other people trying to explain strong vs weak:
- [What To Know Before Debating Type Systems](http://blog.steveklabnik.com/posts/2010-07-17-what-to-know-before-debating-type-systems) by Steve Klabnik, 2010
- [Types for anyone who knows a programming language](https://www.destroyallsoftware.com/compendium/types?share_key=baf6b67369843fa2) by Gary Bernhardt, 2017
