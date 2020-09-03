---
layout:	"post"
title:	"How to Use Modern CMake for an App + Lib Project"
---

* * *

> An example of how to apply some of the so called "Modern CMake" to build a
simple project composed by an executable that uses a library.

Working with build systems is not the easiest task in the Software
Engineering's world, this is especially true for C++ developers where there's
not a "standard" of to use this or that. But things are changing and efforts
towards the development of a standard, or at least a set of guidelines, are
having a positive effect on our daily tasks, aiming to simplify our lives,
cool!

Among the build systems available for C++, one of the most popular is
[CMake,](https://cmake.org/) hold on! Technically, CMake is _not_ a build
system, instead, it is a build system _generator_ whose duty is to generate
the necessary files that will be used by the build system itself (Make, Ninja,
Visual Studio, etc).

To deal with CMake was not as easy as we would like so, but the set of
principles inspired on Modern C++, called Modern (or Effective) CMake has
helped us to develop build systems that are easier to:

  1. Understand
  2. Maintain
  3. Evolve

You can find more detailed information regarding the concepts of modern CMake
on the [CMake's documentation](https://cmake.org/documentation/) and the talks
of [Daniel Pfeifer](https://www.youtube.com/watch?v=bsXLMQ6WgIk) at C++ Now
2017 and [Mathieu Ropert](https://www.youtube.com/watch?v=eC9-iRN2b04) at
CppCon 2017, guys, you rock!

* * *

#### Reasoning About Modules and Dependencies

The main idea of modern CMake is to:

> Instead of reasoning about global flags, structure your project as a graph
of modules with explicit dependencies between these modules.

What?

In the old days of CMake, we are used to inserting a lot of global flags and
includes ( _include_directories_ , __ I'm looking at you) in our
CMakeLists.txt that affects every module, there wasn't any degree of
encapsulation and dependency management between projects was a pain.

Fortunately, things have changed, and now, we're encouraged on think about
modules, for instance: On a project, we have an executable _app_ (a module)
that depends on a library _lib1_ (another module) and we separate what belongs
to _lib1_ 's interface ( **what** it does) and what belongs to _lib1_ 's
implementation ( **how** it does it), more or less, an application of Object
Oriented Programming, or more generally, Modular Programming.

On the CMake jargon, each module gives origin to a **_target_** __ that has a
set of **_properties_** __ (ex: compiler definitions, sources, headers,
libraries, etc).

One of the benefits of this approach is the possibility to hide from the
outside world what is private for the module and what is public and can/should
be used by used by its clients.

#### Target and Properties, remember: Target and Properties

You create a target by invoking a command like these (they seem like a
constructor, right?):

  *  _add_executable_
  *  _add_library_

And you customize a target but modifying its properties like these (they seem
like setters, right?):

  *  _target_include_directories_
  *  _target_compile_definitions_

Each command (there're so many others…) affects one target and can be made
_PRIVATE_ (just used by the module), _INTERFACE_ (just used by clients) or
_PUBLIC_ (used by the module and its clients). Hence, you can have more
granular control over your modules and can encapsulate its characteristics.

#### Example

To illustrate the concepts, let's go to a simple example of C++ project. You
can download the example on this GitHub's repository:
<https://github.com/rvarago/modern-cmake-appAndLib>.

Our project is composed by an executable called _app_ and it uses a static
library called _lib1_ is inside a project's sub-directory. To simplify the
approach, we won't install our library in the OS's standard folder for
libraries, neither write tests for the library nor the executable, but always
remember to write tests for your software! Also, I'm probably not applying all
the concepts of Effective Modern CMake, but I'm struggling to achieve at least
a significant percentage of the guidelines.

The library will export the function: _int sum(const int, const int)_ that
computes the sum of the supplied arguments and returns this value. The
executable will merely invoke this function and print the result on _stdout_.
Fairly simple, just to focus on the CMake 's stuff.

Our project layout is:

![](/assets/img/2018-08-20-how-to-use-modern-cmake-for-an-app-p-lib-project_0.png)

The top-level Makefile will just instruct Make to wrap the creation of the
build folder used to separate building artefacts from the code, the invocation
of CMake and then the compilation of the resultant Makefile by Make again.

Meanwhile, the top-level CMakeLists.txt contains the basic setup for the
project, it defines the CMake's minimum version and adds the _app_ and _libs_
folders as subdirectories by issuing _add_subdirectory_ commands.

* * *

The _libs/CMakeLists.txt_ only purpose is to add each library's folder as
subdirectories of the project, so it will be omitted here. It can be useful on
scenarios where you have many libraries and want to have some feature enabled
for all the libraries, be careful to not expose more than the necessary.

The _libs/lib1/CMakeLists.txt_ defines the _lib1_ target which is a static
library composed by the _lib1_ 's source files by calling:

  *  _add_library(lib1 STATIC ${lib1_SOURCES})_

Also, it include the target _lib1_ 's inclusion directories by calling:

  *  _target_include_directories(lib1 PUBLIC include PRIVATE src)_

Note that the _include_ folder is _PUBLIC_ and will be used by _lib1_ and its
clients, whereas the _src_ folder is _PRIVATE_ and is going to be used just by
_lib1_.

* * *

Finally, the _app/CMakeLists.txt_ defines the _app_ which is an executable
composed by the _app_ 's source files by calling:

  *  _add_executable(app ${app_SOURCES})_

Also, it wires _app_ with its dependency on the _lib1_ target, in this case we
've chosen to set this dependency _PRIVATE_ , so it'll just used by the _app_
target and would be hidden if _app_ had clients. This is done by:

  *  _target_link_libraries(app PRIVATE lib1)_

Look at how easier is to managing dependencies with Modern CMake, a simple
_target_link_libraries_ is enough to link the library, its header, and all
other possibly _INTERFACE_ or _PUBLIC_ transitive dependencies that the
library requires, pretty cool!

#### Conclusion

Build systems for C++ are known to cause sadness, but Modern CMake has come to
the rescue and these days are gradually falling behind. The main concept is to
treat CMake's files as production code and apply the same rules for clarity
and modular design. For CMake, we need to answer three questions?

  1. What are my targets?
  2. What are my target's properties?
  3. How my target is supposed to interact with its clients?

In this way your project will be easier to maintain, your clients will be
happier by using your libraries in an easier manner and everyone wins.

#### References

[1] CMake Documentation. <https://cmake.org/documentation/>

[2] C++Now 2017: Daniel Pfeifer "Effective CMake".
<https://www.youtube.com/watch?v=bsXLMQ6WgIk>

[3] CppCon 2017: Mathieu Ropert "Using Modern CMake Patterns to Enforce a Good
Modular Design". <https://www.youtube.com/watch?v=eC9-iRN2b04>

[4] Effective Modern CMake.
<https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1>

* * *

Originally posted at [Rafael Varago's private
space](https://medium.com/@varago.rafael/how-to-use-modern-cmake-for-an-app-
lib-project-3c2ee6018cde) on August, 20, 2018.


***
*Originally published at https://medium.com/@rvarago*
