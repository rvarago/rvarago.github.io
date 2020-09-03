---
layout:	"post"
title:	"Introduction to CMake for C++"
---

Today, I'm going to talk about how to get started with CMake for building a
C++ application. Firstly, I'll introduce some CMake commands, then we'll test
them with a C++ example.

* * *

> **UPDATE:**

> This article is outdated and the advice given here are no longer considered
good practices, being deprecated in favor of a style known as "Modern CMake"
which has a lot of advantages over the way to use CMake shown here. Please,
consider the following links for introductions on Modern CMake:

[ **How to Use Modern CMake for an App + Lib Project**  
 _An example of how to apply some of the so called "Modern CMake" to build a
simple project composed by an executable…_code.egym.de](https://code.egym.de
/how-to-use-modern-cmake-for-an-app-lib-project-3c2ee6018cde
"https://code.egym.de/how-to-use-modern-cmake-for-an-app-lib-project-
3c2ee6018cde")[](https://code.egym.de/how-to-use-modern-cmake-for-an-app-lib-
project-3c2ee6018cde)

[ **Refactoring a CMake build system**  
 _Some advice to help you when porting a build system based on CMake to use a
modular approach following the so-called
…_code.egym.de](https://code.egym.de/refactoring-a-cmake-build-system-
9898c2030c3a "https://code.egym.de/refactoring-a-cmake-build-system-
9898c2030c3a")[](https://code.egym.de/refactoring-a-cmake-build-system-
9898c2030c3a)

> This article is going to be kept available, since it's part of the CMake's
history, and it may also be useful for showing the advantages of Modern CMake.
Moreover, it may help developers that are working on pre-Modern CMake
projects. However, please consider applying Modern CMake techniques whenever
possible.

Hello,

Today, I'm going to talk about how to get started with CMake for building a
C++ application. Firstly, I'll introduce some concepts regarding CMake, then
we'll run it with a C++ example.

#### Build System

CMake is a cross-platform tool that automates the building process of software
projects.

Normally, a build tool like Make will parse a configuration file ( _Makefile_
) that contains all the commands required to build an artifact based on the
source files and other resources inside the project. In the other hand, CMake
will also parse a configuration file ( _CMakeLists.txt_ ), but instead of
directly build the artifact, it'll generate another configuration file that
will, in fact, build the artifact.

The practice to add another level of indirection to solve a problem is very
common in Computer Science. In the case of CMake, the main advantage is:

> By just one configuration file __ you'll be able to generate different
configuration files to build your project for different platforms (Make,
Visual Studio _,_ etc).

In other words, you are giving portability to your building process.

The configuration generated from _CMakeLists.txt_ that will build your project
based on the **native build system** (Make, Visual Studio, etc) is called
**native configuration file** and the responsible to generate it is called
**generator**. CMake comes with a diversity of generators and you can follow
the references to see the list.

More than portability, by using CMake you can considerably simplify your
building process for projects composed by files spread over multiple sub-
directories, for example, _src_ for   _.cpp_ , _includes_ for   _.hpp_ , build
for generated artifacts, etc.

#### Example

To illustrate the basic structure of a CMake project, let's write an example.

It consists of a C++ application written in Linux named _rvarago-hello-cmake_
that will be compiled against the C++ 14 standard with g++, and it 's composed
of three files: _person.hpp_ , _person.cpp_ , and _main.cpp_. __ The final
result will be an executable named _hello_.

The native build system will be Make, so CMake will generate a _Makefile_
based on a _CMakeLists.txt_.

The project is structured in the following way (output of the _tree_ utility
in Linux):

![](/assets/img/2018-05-30-introduction-to-cmake-for-cpp_0.png)

  *  _build_ is empty now, but it 'll contain the generated _Makefile_ , the executable, and other resources from the building process. The main reason to have this directory is that we can clean all the outputs from CMake just by erasing the contents of this directory
  *  _include_ s for the interfaces ( _.hpp_ )
  *  _src_ for the implementations ( _.cpp_ )

 _person.hpp_ is the interface for the class _Person_ :

 _person.cpp_ is its implementation:

 _main.cpp_ represents the client of _Person_ , by depending on _person.hpp_ :

At this moment, you can compile your project with the following command:

> g++ -o build/hello src/person.cpp src/main.cpp -I./includes -std=c++14

And you'll get the _hello_ executable inside the _build_ directory.

It's also possible to write a _Makefile_ to use Make to conditionally compile
the project, but it won 't be a simple task, because we're dealing with files
spread over multiple directories. This is the scenario where CMake really
shines.

To build our application, represented by the executable _hello_ , based on the
source files, we'll need the following _CMakeLists.txt_ :

To generate the _Makefile_ and other intermediary resources. Firstly, change
to _build_ and call the _cmake_ utility supplying the path to the
_CMakeLists.txt_ , and it'll generate the result inside the calling directory,
that is, _build_ :

> cmake ..

Now, we have the _Makefile_ generated from _CMakeLists.txt_ , and to compile
our application, we need to run make:

> make

And you'll see the _hello_ executable inside the _build_ directory.

#### Conclusion

In this article, we've discussed the basics regarding how to get started with
CMake for automation of building process of C++ applications. We wrote a
simple example that compiled a basic multi-file application and the CMake to
generate the _Makefile_ from _CMakeLists.txt_ required to build the
application with Make as the native build system.

I hope that with this basic overview of CMake you can be able to continue your
studies about how to automate and simplify your building process, maybe adding
[automated unit tests](https://medium.com/@varago.rafael/introduction-to-
google-c-unit-testing-3d564c30f3b0) to your project.

#### References

[1] <https://cmake.org/>


***
*Originally published at https://medium.com/@rvarago*
