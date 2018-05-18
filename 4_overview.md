# Overview

## the language

ES is an *object-oriented* language for performing computations and manipulating objects *within an host environment*.

The host environment will have to provide both the objects described in the ES specification and certain *environment-specific* objects (the description and behavior of which is not indicated in the ES spec).

ES was originally designed to be a *Web scripting* language, but today it is used as a *general-purpose* language.

*A scripting language is a language used to manipulate, customize and automate the facilities of an existing system.*
In this system, functionality is exposed via an API and the scripting language exposes this API to program control.
The system is said to provide an host environment of objects which completes the capabilities of the scripting language.

*A Web scripting language provides mechanisms to make Web pages interactive and perform server computation.*

Since ES is today used in a variety of host environments, the ES spec defines the core characteristics of the language, independent from the particular host environment.

## objects

ES is *object-based*, which means basic language and host facilities are provided by objects.
An ES program is a *cluster of communicating objects*.

**An *object* is a collection of *zero or more properties* each with *attributes* that determine how each property can be used.**

*Properties are containers that hold other objects, or primitive values or functions*.

A *primitive value* is one of the `undefined`, `null`, `string`, `number`, `boolean`, `symbol` *built-in types*.

An *object* is a member of the built-in type `object`.

A *function* is a *callable object*.

A function that is associated to an object via a property is called a *method*.

ES defines a collection of built-in objects:
- *global object*
- objects *fundamentals to the runtime semantics* of the language like:
  - Object
  - Function
  - Boolean
  - Symbol
  - Error
- objects that *represent and manipulate numeric data* like:
  - Math
  - Number
  - Date
- objects for *text processing* like:
  - String
  - RegExp
- objects that are *indexed collections of values* like:
  - Array
  - Typed Arrays (9 different kinds)
- objects that are *keyed collections* like:
  - Map
  - Set
- objects *supporting structured data* like:
  - JSON
  - ArrayBuffer
  - SharedArrayBuffer
  - DataView
- objects *supporting control abstractions* like:
  - GeneratorFunction
  - Promise
- *reflection objects* like:
  - Proxy
  - Reflect

ES also defines a set of built-in *operators*.

Large ES programs are supported by *modules* which allow programs to be divided into multiple sequences of statements and declarations.
Each module specifies the declarations which it uses and are provided by other modules and also its own declarations which are available to other modules.

ES syntax is *relaxed* in the sense that there is no need to declare the type of a variable nor are types associated with properties, and calls to functions do not required the function declaration to appear textually before them.

## prototype and delegation

ES include syntax for class definitions, but ES objects *are not class-based*.

*Objects can be created in various ways* including by literal notation or via *constructors* which create objects and initialize all part of them by assigning initial values to their properties.

*Each constructor is a function* which has a property named *prototype* that is used to implement *prototype-based inheritance* and *shared properties*.

To create objects, constructors must be called in `new` expressions.
Invoking a constructor without `new` has consequences based on the specific constructor.

Every object created by a constructor has an *implicit reference* (its `[[Prototype]]`) to the value of its constructor `prototype` property.

The series of links created by the internal `[[Prototype]]` of objects forms the *prototype chain*.

**When a reference is made to a property in an object that reference is to the property of that name in the first object `[[Prototype]]` chain that contains a property of that name.**

First, the object mentioned is directly examined for such property name, if the object contains such property, then that is the property to which the reference refers. If the object doesn't contain the property name, the `[[Prototype]]` for that object is examined next, and so on.

In a traditional class-oriented language, state is carried by the instances, methods are carried by the classes and inheritance is for structure and behavior.
In ES, state and methods are carried by objects. Structure, behavior and state are all inherited.

*All objects that don't directly contain a particular property that their `[[Prototype]]` contains, share that property and its value.*

Properties can be added to objects dynamically, by assigning values to them. So constructors are not required to name or assign values to the constructed objects properties.

The ES built-in objects follow a class-like pattern of constructor functions, prototype objects and methods.

Class syntax allow programmers to define objects that conform to this same class-abstraction.

## ES strict variant

The ES spec defines a *strict variant* of the language, usually called *strict mode*.

This variant excludes some syntactic and semantic features and modifies some others. It also specifies additional error conditions reported by exceptions.
