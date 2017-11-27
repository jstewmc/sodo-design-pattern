# Service object, Data object (SODO) Design Pattern

Use a swarm of tiny components to create flexible applications.

## Introduction

There exists a time in every programmer's life when they question their life decisions: inheriting legacy code. 

Legacy code is a mess: logic is repeated; control is everywhere; technology is outdated; decisions are undocumented; and, objects are ambiguous. Legacy code is smelly, spaghetti code. 

However, it wasn't always so. At some point in its life, all _legacy code_ was _greenfield code_, and some developer had the best of intentions to build things the right way (or, at least, to return and fix the issues). 

The transition from _greenfield code_ to _legacy code_ is the nature of software. 

Code doesn't exist in a vacuum. It exists within the context of resource constraints, deadlines, business rules, skill levels, customer needs, product decisions, specification changes, and much more.

Like _physical engineers_ who create bridges and buildings, _software engineers_ create expensive virtual structures which, once laid down, are difficult to change. 

And, like physical structures, virtual structures age. New solutions stack upon old ones; small, quick changes accumulate; team members leave; technologies evolve; best practices shift; priorities change; and, soon enough, someone's proud accomplishment (or embarassing solution) becomes someone else's legacy code.

There simply is no way to prevent it. As soon as software is released, it exists. And, things that exist are resistant (and expensive) to change. 

Instead of lamenting _legacy code_, which is the ultimate destination of any _released code_, we must prevent it. We must build applications that are flexible, capable of solving _present problems_ without sacrificing the freedom to solve _future problems_ as well. 

The key to a flexible application is nanoscopic components. An application created from a swarm of tiny, decoupled, reusable, tested, injectable components can change to meet any requirement. 

To that end, I propose the Service Object, Data Object (SODO) Design Pattern.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

### Data objects (aka, "data")

A data object is a virtual representation of a person, place, or thing (e.g., a user).

* A data object MUST be a newable object.
* A data object MUST be passed _required_ data via its constructor, if required data exists (1).
* A data object MUST be passed _optional_ data via setter methods (1).
* A data object MUST NOT provide methods that alter its state. Don't ask bread to bake itself (4).
* A data object SHOULD provide public getters for its non-secret attributes.
* A data object SHOULD define separate child-objects for each possible state (5).
* A data object MAY provide inspecters  (e.g., `has*()` and `is*()`) (2). 
* A data object MAY provide verifiers (e.g., `verify*()` (3).
* A data object MAY be defined in a separate repository (6).


### Service objects (aka, "services")

A service object is an action (e.g., create-user, confirm-user, archive-user, etc). 

* A service object MUST be stateless. A single, shared instance should serve the entire application.
* A service object SHOULD be named <verb><data> (12). 
* A service object MUST be passed _required_ dependencies via its constructor, if required dependencies exist (7). 
* A service object MUST be passed _optional_ dependencies via setter methods (7). 
* A service object SHOULD NOT accept more than three constructor arguments (8).
* A service object MUST define an interface for each dependency, if the language supports interfaces. 
* A service object MAY define concrete implementations of its interfaces to use as defaults.
* A service object SHOULD be defined in a service manager.
* A service object SHOULD NOT accept its own service manager as a dependency (9).
* A service object SHOULD NOT expose more than one public, instance method (10, 13).
* A service object MUST NOT expose public properties. 
* A service object MAY have unlimited private methods and properties (11).
* A service object MAY be defined in a separate repository (6).


### Service manager (aka, "framework")

A service manager stores (and retrieves) service definitions and instances.

* A service manager MUST identify services by name. 
* A service manager MUST provide an environment-aware, application-level configuration array.
* A service manager MUST support building the configuration array from separate files.
* A service manager MUST support defining services in separate files.
* A service manager MUST provide service definitions with the configuration.
* A service manager MUST provide service definitions with itself.
* A service manager MUST cache an instantiated service for the application's lifecycle.


## Discussion

Additional notes about some specifications:

1. Data is either _required_ or _optional_. Required data is an attribute without which an object cannot exist. For example, a user must have an email address (i.e., it's required), but they may or may not have a phone number (i.e., it's optional).
2. Introspective methods provide information about an object's attributes without changing them. These are commonly prefixed with `has` or `is`, but follow your preferred convention. 
3. Data objects may possess secret attributes like passwords, reset tokens, etc. Instead of providing a get method, which could publicly expose the secret, it is best practice to expose `verify*()` methods, which when given a value, return true or false.
4. Arguably, the most important thing about data objects is their simplicity. They hold data. They do not do anything else. They are exceedingly dumb. Don't ask bread to bake itself!
5. Data objects may transition state. For example, a user might transition from _created_, to _confirmed_,  to _deactivated_, to _deleted_. Representing separate states as separate child objects is a powerful pattern. Extending a parent class to create each child allows all states to inherit common properties and methods while providing their own custom properties and methods as needed.
6. Defining data objects and services in separate repositories can be cumbersome but incredibly flexible. Separate repositories allow separate semantic versioning, test suites, and code coverage analysis. The same data object or service object may be used anywhere across multiple applications throughout an organization or community.
7. Like a data object's data, a service object's dependencies can be divided into _required dependencies_ and _optional dependencies_. A _required dependency_ is a service or configuration option without which a service cannot perform its function.
8. A constructor with more than three arguments is an indication that the service is doing too much and should be broken into smaller nested services.
9. When a service defined in a service manager accepts the same service manager as a dependecy, a circular reference is created. Some languages have difficultly de-referencing circular references and freeing memory. In short-running processes like responding to an HTTP request, this is not a problem. However, in long-running processes such as message workers or large test suites, this can quickly become a memory (and performance) problem. 
10. Ideally, the service exposes a single public method that allows it to be treated as a function such as PHP's magic `__invoke()` method or Ruby's conventional `call()` method. The method's arguments should be invocation-level arguments. Any argument that can be used on multiple invocations should be constructor-injected instead.
11. An abundance of private methods and properties is an indication the service is doing too much and should be broken into small nested services. Remember, a service does _one_ thing and it does it well. If a service's _one_ thing involves mutliple smaller things, you might want to create separate services for each smaller thing.
12. In English, I recommend using hyphens to separate the service's name when used in discussion (e.g., "the parse-foo service"). In code, use your language's standard class naming convention (e.g., in PHP, `class ParseFoo`; or, in Ruby,  `class ParseFoo`, etc).
13. If the service does expose multiple public methods, each method should perform a small variation of the service's main function, not something new. For (a self-centered) example, my [detect-environment](https://github.com/jstewmc/detect-environment) service detects the application's environment, and it exposes multiple public methods like `isDevelopment()` and `isProduction()` to make it easier for developers to check the current environment. All the public methods are variations of the same theme, detecting the application's environment.

## Version 

This standard adheres to the [semantic versioning](https://semver.org) standard:

### Version 0.1.0, November 24, 2017

* Initial release 

## About

The Service Object, Data Object (SODO) Design Pattern is authored by [Jack Clayton](https://github.com/jstewmc). 

If you'd like to leave feedback, please [open an issue](https://github.com/jstewmc/sodo-design-pattern/issues/new) or submit a pull request.
