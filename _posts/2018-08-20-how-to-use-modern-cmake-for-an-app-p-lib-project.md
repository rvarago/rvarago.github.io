---
layout: "post"
title:  "How to Use Modern CMake for an App + Lib Project"
tags:   build-system cmake c++
---

> An example of how to apply some of the so-called "Modern CMake" to build a simple project composed by an executable that uses a library.

* * *

# Introduction

Working with build systems is not the easiest task in the Software
Engineering's world, this is especially true for C++ developers, where there's no widely accepted "standardized" process, yet. However, things are changing and efforts towards the development of a standard or, at least a set of guidelines, are having a positive effect on our daily tasks, aiming to simplify our lives, cool!

Among the build systems available for C++, one of the most popular is [CMake,](https://cmake.org/) hold on! Technically, CMake is _not_ a build system, it's rather a build system _generator_, whose duty is to generate the necessary files that will be used by the build system itself (Make, Ninja, Visual Studio, etc).

Dealing with CMake has not been as easy as we would like it to be, but the set of principles inspired by Modern C++, called Modern (or Effective) CMake has helped us to better structure our build systems, making them easier to:

  1. Understand.
  2. Maintain.
  3. Evolve.

You can find more detailed information regarding the concepts of modern CMake on the [CMake's documentation](https://cmake.org/documentation/), and the talks given by [Daniel Pfeifer](https://www.youtube.com/watch?v=bsXLMQ6WgIk) at C++ Now
2017 and [Mathieu Ropert](https://www.youtube.com/watch?v=eC9-iRN2b04) at CppCon 2017. You guys rock!

# Reasoning About Modules and Dependencies

From my perspective, modern CMake might be summarized as:

> Instead of reasoning about global flags, prefer to structure your project as a graph of modules with dependencies between them being EXPLICIT.

What?

In the old days of CMake, we were used to adding a lot of global flags and directory-scoped commands (`include_directories` and friends, I'm looking at you) in our `CMakeLists.txt` that affected many more modules than it should have, there was no insulation and dependency management between projects was painful.

Fortunately, things have improved. We're now encouraged to think about modules, for instance: On a project, we have an executable `app` (a module)
that depends on a library `lib1` (another module), and we separate what belongs
to `lib1`'s interface (**what** it does) from what belongs to `lib1`'s implementation (**how** it does), that is, more or less, Modular Programming applied to build systems.

On CMake's jargon, each module gives origin to a **target**, which has a set of **properties** (e.g.: compiler definitions, sources, headers, libraries, etc).

Among the benefits of this approach, we can hide private details of a module from the outside world, so that clients don't see and shouldn't care about.

# Target and Properties

We create a target by invoking a command like these (they do seem like a constructor, right?):

  *  `add_executable`.
  *  `add_library`.

And we customize a target by modifying its properties like these (they do seem like setters, right?):

  *  `target_include_directories`.
  *  `target_compile_definitions`.

Each command (there're so many others…) affects a single target and can be made `PRIVATE` (only used by the module), `INTERFACE` (only used by clients), or `PUBLIC` (used by the module **and** its clients). Hence, we can have much more granular control over modules and thus hide its private details.

## Example

To illustrate the concepts, let's go over an example of a C++ project.

The latest version of the example can be obtained [here](https://github.com/rvarago/modern-cmake-appAndLib).

Our project is composed of the executable `app` and it depends on the library `lib1`. To simplify the approach, we won't install our library on the operating system, nor write tests, but please write tests for your software :-)! Moreover, chances are high that I might not be following all
the advice that is given by Effective Modern CMake, but striving to achieve a significant percentage of the guidelines, at least.

The library will export the function `int sum(const int, const int)`, which computes the sum of its arguments and returns the result. The
executable will then invoke this function and print the result to the console. Fairly straightforward, the goal is to focus on CMake stuff.

The project's layout looks like this:

![](/assets/img/2018-08-20-how-to-use-modern-cmake-for-an-app-p-lib-project_0.png)

The top-level `Makefile` will simply instruct Make to wrap helper commands to drive CMake.

Meanwhile, the top-level `CMakeLists.txt` has the basic setup for the project. It's where we define the minimum version of CMake that we expect, and add the `app` and `libs` subdirectories by issuing `add_subdirectory` commands:

<script src="https://gist.github.com/rvarago/9d054acd4eaddb7ec58e28165857cdf4.js"></script>

The sole purpose of `libs/CMakeLists.txt` is to group all libraries and it won't be mentioned further. It may be useful when we have many libraries and want to have features enabled for all of them, but be careful and don't expose more than what is necessary.

The `libs/lib1/CMakeLists.txt` defines the `lib1` target:

  *  `add_library(lib1 src/lib1-priv-impl.cpp)`.

It includes the target `lib1`'s path to includes:

  *  `target_include_directories(lib1 PUBLIC include PRIVATE src)`.

Notice that the `include` directory is `PUBLIC`, so will be used by `lib1` and its clients, whereas `src` is `PRIVATE`, so it will be used only by `lib1` itself:

<script src="https://gist.github.com/rvarago/5e2b60ab92939c61881e56d4dce65c2e.js"></script>

Finally, `app/CMakeLists.txt` defines the executable `app` composed of `app`'s source files:

  *  `add_executable(app src/main.cpp)`.

It also wires `app` with its dependency on `lib1`. In this case, we've chosen to set this dependency `PRIVATE`, so it'll only be used directly by `app` target and would be hidden if `app` had clients (not usual for executables, though). This is done by:

  *  `target_link_libraries(app PRIVATE lib1)`.

<script src="https://gist.github.com/rvarago/156c5b4cdb3816eabb3755633f821d87.js"></script>

Look at how easier it is to manage dependencies with Modern CMake compared to the old approach. A mere `target_link_libraries` is enough to link against the library, get access to its header, and all other transitive dependencies expressed by `INTERFACE` or `PUBLIC`requires, pretty neat!

# Conclusion

Build systems for C++ are well-known to be complex and rather hard to understand, but CMake has become easier to use than before.

The idea is to simply write CMake's build scripts with the same level of care as we write production code, and thus focus on clarity and modularity. In CMake, we need to answer these three questions:

  1. What are my targets?
  2. What are my target's properties?
  3. How should my target interact with its dependencies and clients?

By thoroughly thinking about these questions your build system should become easier to maintain, and easier to be consumed by clients.

# References

[1] [CMake Documentation](https://cmake.org/documentation/).

[2] [C++Now 2017: Daniel Pfeifer "Effective CMake"](https://www.youtube.com/watch?v=bsXLMQ6WgIk).

[3] [CppCon 2017: Mathieu Ropert "Using Modern CMake Patterns to Enforce a Good Modular Design"](https://www.youtube.com/watch?v=eC9-iRN2b04).

[4] [Effective Modern CMake](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1).

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
