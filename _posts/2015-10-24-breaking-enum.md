---
layout: post
published: false
title: Breaking Enum
---

Breaking changes comes in many ways.

Tipically we're used to think of breaking changes when we change method signatures, classes, return types or public interfaces. We as programmers know that we should avoid breaking changes because that can affect every little piece of code that depends on the changed part.

While they must be avoided, they're normal part of code evolution. New features sometimes need big changes. Semantic versioning give us some helper lines in order to manage expectations on the library consumers wheter a new version might include or not breaking changes. Usually only on major versions increments a library can include them, but even so, they're should be avoided. There's always some "disapointment" when we have to re-write our code that was already working in order to adapt to a new library version.

## Enougth if the talking...

Imagine that we have an application that consists on a core library (named BreakingEnum.Common.Dll on this example), another helper library that can be a third party plugin (named BreakingEnum.Helper.Dll) and a frontend aplication, that puts it all in motion. All of this components are only referenced by the libraries and not by source, as they are provided by different providers.