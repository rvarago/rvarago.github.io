---
layout: "post"
title:  "Refactoring a CMake Build System"
tags:   build-system cmake c++
---

> A piece of advice that may help when porting a build system based on CMake to use a modular approach following the so-called Modern CMake set of practices.

* * *

|![CMake.](/assets/img/2019-01-10-refactoring-a-cmake-build-system_0.png)|
|:--:| 
| *CMake. Source: <https://commons.wikimedia.org/wiki/File:Cmake.svg>.*|

This is a follow-up of [How to Use Modern CMake for an App + Lib Project.]({{ site.baseurl }}{% link _posts/2018-08-20-how-to-use-modern-cmake-for-an-app-p-lib-project.md %}).

* * *

At times, the build system isn't the easiest, nor the most exciting, aspect of a software project. On a high-level, the build system describes the topology of the project, its modules and, more critically, how modules relate to each other.

In the end, we end up with a graph, namely a directed graph. Nodes map to modules (_target_ ) with a set of features (_properties_), while edges map to relationships (_libraries_): target A depends on targets B and C.

The graph starts to become trickier when we introduce transitive
dependencies. That is the dependencies of your dependencies:

    If A depends on B and B depends on C
      Then A depends on C 

The question is:

As target A, should I handle (or care about) B and C? Or Should I handle B and let B handle C on my behalf? 

I'd rather say let B handle C. Otherwise, the burden to reuse code tends to be higher than it already is. Think about it: If B depends on C, D, E, …, Z, and to use B, A needs all of them, should A handle all these targets again
(presumably, B has already handled them).

Build systems have been using different approaches to manage this complexity: ranging from strict rules that must be abode by, to the flexibility that allows Engineers to solve each problem differently. CMake is flexible, really flexible.

Generally, I like flexibility, but as the coolest superhero's
uncle once said: **"with great power comes** **great responsibility"**.
Here, as you might've guessed: flexibility means power. And without guidelines and standards, it may be hard to maintain flexible build systems. Moreover, it's harder to onboard new colleagues into different projects, since each project follows a particular, and sometimes completely different, way to do things. There are so many ways of doing things, after all

Currently, as far as I know, CMake offers three ways to handle dependencies:

  1. Copy and paste the source code for each dependency into your final target.
  2. Make every target dependent on all other targets by using `link_libraries` and global variables.
  3. Treat each target's dependency as the target's property, with proper visibility (see above) by using `target_link_libraries` and let CMake resolve the transitive dependencies for us.

To my mind, Option 3 sounds better: it's modular, avoids duplication, and doesn't bloat targets with unnecessary dependencies.

However, due to historical reasons, we've been using options 1 and 2 more often than not in CMake-based projects.

Since CMake 3.0.0, Option 3 has become available, and gradually projects have been ported to benefit from the modular approach offered by Option 3. The task isn't that easy, though. Refactoring requires care.
We don't want to break what already works. But, if done mindfully, it can
be quite fun and yield many benefits.

Roughly speaking, that's Modern CMake in a nutshell. It's all about Information Hiding.

We have to be explicit to distinguish between the properties required for the **target** itself and the properties required for the target's **clients**:

  * If it's only used internally by the target, then `PRIVATE`.
  * If it's only used by clients, then `INTERFACE`.
  * If it's used internally by the target and clients, then `PUBLIC`.

By property, I mean generally: compiler options/flags, inclusion directories, dependencies, etc.

## Refactoring a CMake-Based Build System

After discussing the "Why" and "What". We shall now cover some advice based on my previous experiences. Hopefully, it will help you through your journey porting build systems into Modern CMake. Please, bear in mind that those are advice based on **my** experience, not
unbreakable rules. They may or may not make sense to your project.

### Have a Proper CI Infrastructure Inplace

I'd say that this always important, regardless of what you're doing or
planning to do, i.e. not only for build systems.

A CI will let you be more confident about your refactoring.

When porting part of a build system, we're changing significant attributes of our project's structure. We can easily forget that some library only exists on our machine, and not on the production environment. Further, such a library could have been built using particular attributes tied to our local machine (compiler, flags, etc).

Having a proper CI building our project against the production
configuration, running automated tests, and so on, reduces the chance of changing the exiting behaviour (since we're refactoring, we tend to preserve the existing behaviour).

Furthermore, it's a good idea to have a development environment that is
reproducible and compatible with the production environment. Containers might help us here.

###  Bump CMake's Version to ≥3.0.0

Most of the required features only exist starting from version 3.0.0. I'd say
that we should assess the possibility to use an even more up-to-date version than 3.0.0 (latest?), so we can benefit from new features, bug fixes, etc. For instance, `target_sources` has made it into CMake at 3.1.0.

###  Avoid Refactoring the Source Code

The goal is to refactor the build system and not the code that it builds. If we do both, we might end in a state where we don't know what has caused some change in the behaviour. Given that when porting a build system, we may visit plenty of source code, we may very well discover possibilities to improve in the code too.

We want to go one step at a time.

Only immediately change the code if it's necessary. If that is the case, then at least consider doing it in separate commits. Otherwise, postpone for a future iteration.

###  Start Small and Keep it Going Gradually

This advice is not new or even exclusive to build system refactoring. The idea is to keep things under control, yet changing the build system
gradually through a sequence of well-known states over time.

Resist the temptation of porting the whole build system at one single and gigantic step. Instead, prefer doing it target by target.

Start with the simplest target possible. Ideally, one that doesn't depend on any other project's target. Port it, understand the process, feel comfortable, and then repeat the process to the other targets, dealing with their complexities as they manifest.

###  When Porting a Target, Consider its Visibility

Visibility has a huge impact on the dependency graph. Not all the target's
properties should be visible to clients. For instance, say a target A uses some other target B, but B is not required by C, which is a client of A.

That is useful not only for dependent targets but also for compiler options and other properties. For example, say that the private implementation target A requires C++17, but this requirement is not visible on its
public API, so A's clients don't need a compiler with support for C++17
in order to use A.

### Do not Pay Much Attention to the Dependency Graph Just Yet

When porting a target, we might be lured to introduce an enormous impact on the whole dependency graph. That might not be a good idea.

We might be better of focusing on the target itself and letting its dependent targets to be ported in a second step. Firstly, our goal is the port the targets and then simplify the dependency graph.

If we face a scenario where target A depends on a target B that is under our control, but we don't need to port B to port A, let B be ported
later. Otherwise, we might end in a loop, and nothing will be ported at all.

###  When Facing Cycles, Consider Extracting the Common Code into a Library

Sometimes we face scenarios where A depends on B, and B depends on A, this is called a cycle. It may hint for bad design decisions that were made in the first place.

In some cases, we may be tempted to copy and paste source files from one target to the other and then "get rid" of the explicit dependency.  That's most likely a violation of a Don't Repeat Yourself (DRY). A few cons of this approach:

  1. Duplicated representation of a piece of knowledge leads to multiple "source of truths", and thus to inconsistencies.
  2. We need to maintain both sides of the code.
  3. Slower compilation times, because the compiler needs to build the same code twice, instead of once and then link it twice.

When that happens, we should analyze the part of both libraries A and B that is common. And then perhaps extract it into another library C, letting A and B depending on C. Often, the process might result in multiple libraries C, D, and so on, but that depends on how the concerns are split, etc.

It's important to mention that this is sort-of a violation of a previous guideline, where we said that we should avoid touching the source code. Granted, this should be a last resort measure, but sometimes it's unavoidable. We should rely on our judgement.

### After Porting a Few Libraries, Gradually Start to Think About the Dependency Graph

After porting some libraries, we'll have more knowledge about the project structure, how the dependencies are connected, etc. At that time, we might start to think about the dependency graph in a broader sense, simplifying unnecessary dependencies, altering libraries' visibility level (if it makes sense), etc.

### Avoid Macros and Commands that Affect the Global Scope

 Frequently, we write "helper" macros that link all targets to a "common set of properties", and then each client customizes that as it wishes.

But this may not scale.

First of all, not all libraries might need (or will always need) this "common set of properties". That may lead to a much more complex dependency graph than it had to be, simply because we'd guessed that all the targets need some flags, libraries, etc. And then we realized that this is not true.
Moreover, the visibility level for each common property might
be different among targets. Instead, we should be explicit about what each target requires.

That is especially true for commands that affect global/directory scope, e.g. `include_directories` and `link_libraries`. They make our build system
harder to understand and maintain, as they may add properties that we don't need and, more subtle, we might lose the ability to answers like: "Where did this dependency come from?", "How did this come to here?".

Prefer the target-scoped counterpart of those commands, e.g. `target_include_directories`, `target_link_libraries`.

## Conclusion

Porting a whole build system to use Modern CMake can be a challenging task. It requires effort and discipline and more importantly: a continuous effort and discipline. Otherwise, we can easily come back to a global-based approach and lack of structure.

At first, it might not sound useful, but that changes once we see our
dependencies being handled properly, our project becomes more straightforward to reason about and therefore evolve.

All of this leads to code that is easy to reuse code. That is in contrast to the scenario where it appears to be nearly impossible to import a library without having to cope with plenty of other libraries that we don't even care.

I hope that this post will help you in the quest towards Modern CMake and libraries that we can easily share and consume.

Bear in mind: refactoring requires discipline to keep things under
control, especially when we have so many moving parts.

## References

[1] [CMake Documentation.](https://cmake.org/documentation)

[2] [C++Now 2017: Daniel Pfeifer "Effective CMake"](https://www.youtube.com/watch?v=bsXLMQ6WgIk)

[3] [CppCon 2017: Mathieu Ropert "Using Modern CMake Patterns to Enforce a Good Modular Design".](https://www.youtube.com/watch?v=eC9-iRN2b04)

[4] [Effective Modern CMake.](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1)

[5] [How to Use Modern CMake for an App + Lib Project.]({{ site.baseurl }}{% link _posts/2018-08-20-how-to-use-modern-cmake-for-an-app-p-lib-project.md %})

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*