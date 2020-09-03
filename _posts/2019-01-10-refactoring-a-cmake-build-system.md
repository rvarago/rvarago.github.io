---
layout:	"post"
title:	"Refactoring a CMake build system"
---

* * *

![](/assets/img/2019-01-10-refactoring-a-cmake-build-system_0.png)

CMake Icon. Source: <https://commons.wikimedia.org/wiki/File:Cmake.svg>

> Some advice to help you when porting a build system based on CMake to use a
modular approach following the so-called Modern CMake.

Abuild system isn't the easiest part of a Software Engineering project. On a
high-level analysis, the build system describes the project, its content and
more critically: how project modules are related to each other.

In the end, you have a well-known data structure named graph, specifically a
directed graph. Where each node is a project's module (in CMake: _target_ )
with a set of features (in CMake: _properties_ ), and edges (in CMake:
_libraries_ ) that state how the nodes are related: target A depends on
targets B, C, and D...

The graph starts to become even more complex when you have transitive
dependencies (aka: the dependencies of your dependencies), like:

    
    
    A => B  
    B => C  
    Hence, A => B => C, i.e, **A = > C** 

The question is:

As target A, should I handle B and C? Or Should I only handle B and let B
handle C for me?

I'd prefer to let B handle C. Otherwise, the burden to reuse code tends to be
harder than it already is. Think about it: If B depends on C, D, E, …, Z, and
to use B, A needs all of them, should A handle all these targets again
(because B has already handled them).

Build systems have been using different approaches to manage this complexity:
from strict rules that must be followed to make it work, to flexibility that
allows Engineers to solve each problem differently. And CMake is flexible.

Particularity, I'm a fan of flexibility, but as the coolest Marvel superhero's
uncle once said: ** "with great power comes** **great responsibility "**.
Here, as you might've guessed: power means flexibility. And without guidelines
and standards it can be hard to obtain maintainable build systems; moreover,
it's hard to onboard new colleagues into different projects since each project
follows a particular, and sometimes completely different, way to do things.

Currently, as far as I know, CMake offers three ways to handle dependencies:

  1. Copy and paste the source code for each dependency into your final target
  2. Make every target depends on all the other targets by using _link_libraries_ and global variables, even if some of them don't really need
  3. Treat each target's dependency as a target's property, with the right visibility (see above) by using _target_link_libraries_ and let CMake solves the transitive dependencies for us

In my opinion, option 3 sounds much better: avoids code duplication (1) and
doesn't bloat targets with unnecessary dependencies (2).

But unfortunately, historically we've been using options 1 and 2 in our CMake-
based projects. Since CMake 3.0.0, option 3 has become available and projects
have been ported to benefit from the modular approach offered by using option
3. The task isn't that easy, refactoring always requires care to avoid
breaking what's already working. But, in my opinion, if done carefully, it can
be very fun and yields great benefits.

Roughly speaking, that's Modern CMake. It's all about Information Hiding.

You need to be explicit to distinguish between properties required for the
target itself and properties required for its clients:

  * If it's only used internally by the target (in CMake: _PRIVATE_ )
  * Or only used by clients when transitivity happens (in CMake: _INTERFACE_ )
  * Or for both cases (in CMake: _PUBLIC_ )

By property, I mean: compiler options/flags, inclusion directories,
dependencies, etc.

### Refactoring a CMake-based build system

OK, since we've already discussed "Why" and "How".

Let's now talk about some advice that might help you through your journey to
port your project's build system to use Modern CMake that has been used in my
team, yielding satisfactory results.

Also, please, bear in mind that those are advice based on my experience, not
unbreakable rules. They may or may not make sense to your project.

[Here](https://code.egym.de/how-to-use-modern-cmake-for-an-app-lib-project-
3c2ee6018cde) you can find an explanation and example of a project that
follows some Modern CMake guidelines.

* * *

####  **First of all, have a proper CI configured**

I'd say that this always important, regardless of what you're doing or
planning to do.

A proper CI will let you be more confident about your refactoring.

When porting part of your build system, you're changing significant attributes
of your project's _internal_ structure. You can easily forget that some
library only exists on your machine, and not on the production environment; a
library was built using some particular attributes compatible with your
machine; or that some compiler flag is different, etc.

Having a proper CI that builds your project against your production's
configuration, runs the automatic tests (please, don't tell me that you don't
have automatic tests…). Therefore, reduce the likelihood of changing the
behavior; since you're refactoring, you don't want to change the behavior.

Furthermore, it's always a good idea to have a development environment
reproducible and compatible with your production environment. Containers might
help you here.

* * *

####  **Increase to CMake version: ≥ 3.0.0**

Most of the required features only exist in version 3.0.0 and so on. I'd say
that you should assess the possibility to use a higher version than 3.0.0, so
you can benefit from new useful features that have been merged into CMake. For
instance, _target_sources_ has been available since 3.1.0.

* * *

####  **Avoid refactoring the source  code**

The goal is to refactor the build system, not the code it builds. If you do
both, you might end in a state where you don't know what has caused some
change in the behavior. Given that when porting a build system, you may visit
a lot of source code internals, you may discover possibilities to improve.

But only change the code if it's absolutely necessary. If it is, consider
doing it in separate commits.

* * *

####  **Start small and keep it going gradually**

This advice isn't new or exclusive for build system related refactoring. But
the idea is to keep things under control, yet changing the build system
gradually through well-known states over time.

Don't attempt to port the whole build system at once. Instead, do it target by
target over time. Start by selecting the simplest target; ideally, one that
doesn't depend on any other project's target. Port it, understand the process,
and then replicate to the other targets, dealing with their complexities as
they appear.

* * *

####  **When porting a target, think about visibility**

Visibility has a major impact on the dependency graph. Not all target's
properties should be visible to clients. For instance, a target A use some
other target B, but B isn't required by C that is a client from A.

This is useful for dependent targets, but also for compiler options. For
instance, target A implementation requires C++17, but it's not visible in its
public API, so A's clients don't need to have a compiler that supports C++17
to use A.

* * *

####  **Do not pay much attention to the dependency graph at  first**

When starting to port a target, you might be tempted to already make an impact
on the whole dependency graph. Don't do it. Focus on the target itself, let
the target's dependent targets be ported after. In the beginning, the goal is
the port the targets and then simplify the dependency graph.

If you face a scenario where target A depends on a target B that is under your
control, but you don't need to port B in order to port A, let B be ported
later. Otherwise, you might end in a loop, and anything will be ported at all.

* * *

####  **When facing cycles, consider extracting the common code to a
library**

Sometimes you might face a scenario where A depends on B, and B depends on A.
It's called cycle. Sometimes, it indicates a bad design decision.

In some cases, you may be tempted to copy and paste some source files from to
the other and "remove" the dependency. But it's probably a violation of a
well-known principle: Don't Repeat Yourself (DRY). Some cons of this approach:

  1. Duplicated representation of a piece of knowledge leads to multiple "source of truths", and so to inconsistencies
  2. You need to maintain both sides of the code
  3. Slower compilation because the compiler will need to build the same code twice, instead of once and then link it twice

When it happens, analyze the part of both libraries A and B that is common,
then extract it to a common library C and let A and B import C. Sometimes, it
might result in multiple libraries C, D, etc. depending on how the concerns
are distributed.

* * *

#### After porting some libraries, gradually start to think about the
dependency graph

After porting some libraries, you'll have more knowledge about your project's
structure, how the dependencies are used, etc. At that time, you might start
to think about the dependency graph in a broader sense, simplifying
unnecessary dependencies, making some libraries' visibility level to PUBLIC if
it makes sense, etc.

* * *

#### Get rid of macros and commands that affect the global scope

To help us, we sometimes write macros that link all targets to a "common set
of properties" and then each client customizes it as it wishes.

But it may be the wrong choice.

First of all, not all libraries might need, or won't need, the common
properties. This can result in a much more complex dependency graph than it
needs to be because we'd guessed that all the targets may need some flags,
libraries, etc. Moreover, the visibility level for each common property might
be different among targets. Instead, be explicit about what each target needs.

This is especially true for commands that affect global/directory scope. Like
_include_directories_ , _link_libraries_ , etc. They make your build system
harder to maintain, because they may add properties that you don't need and
more subtle, you might lose the ability to answer this question when analyzing
your dependency graph: "Where/How did this dependency come from?".

Always prefer the target-scoped version of those commands, like **_target_**
__include_directories_ , **_target_** __link_libraries_ , etc.

### Conclusion

Porting a whole build system to use Modern CMake can be very challenging, it
requires efforts and discipline; more importantly: continuous effort and
discipline. Otherwise, you can easily come back to a global-based structure.

At first, it might not sound very useful, but it changes once you see your
dependencies being handled properly, your project easier to reason about and
simplify. Hence how easier is to reuse code now, compared to before where it
was nearly impossible to import a library without dealing with many other
libraries that you don't, or shouldn't, care.

I hope that article helps you in your journey towards Modern CMake.

Always bear in mind: refactoring requires discipline to keep things under
control.

### References

[1] CMake Documentation. <https://cmake.org/documentation/>

[2] C++Now 2017: Daniel Pfeifer "Effective CMake".
<https://www.youtube.com/watch?v=bsXLMQ6WgIk>

[3] CppCon 2017: Mathieu Ropert "Using Modern CMake Patterns to Enforce a Good
Modular Design". <https://www.youtube.com/watch?v=eC9-iRN2b04>

[4] Effective Modern CMake.
<https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1>

[5] How to Use Modern CMake for an App + Lib Project. <https://code.egym.de
/how-to-use-modern-cmake-for-an-app-lib-project-3c2ee6018cde>

