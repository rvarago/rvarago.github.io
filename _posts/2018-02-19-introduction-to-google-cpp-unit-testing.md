---
layout: "post"
title:  "Quick Look at GoogleTest for C++"
tags:   tests c++
---

> A quick look at GoogleTest library for automated testing in C++.

* * *

# Installation

First, you need to install GoogleTest, which can be done by following the instructions at [https://github.com/google/googletest](https://github.com/google/googletest).
    
# Example

This example will be a simplified, and poorly written, calculator for signed-integers (a-ha!), "suitably" named `calc`, and
it supports addition and subtraction.

The header file `calc.hpp`:

<script src="https://gist.github.com/rvarago/3cfb9a93c83ba114660ac61748a7d8ca.js"></script>

The implementation file `calc.cpp`:

<script src="https://gist.github.com/rvarago/9696e1af67ce6052b8aeddf0b4c0d9d7.js"></script>

And the test file `calc_test.cpp`:

<script src="https://gist.github.com/rvarago/455d424f63c5e9ed1819568185c032be.js"></script>

There is some stuff from GoogleTest:

  1. Its header file `gtest/gtest.h`.
  2. The `TEST` macro creates the test cases, where the first parameter is the test-suite name and the second is the test-case name.
  3. The `ASSERT_EQ`, member of the `ASSERT_*` group of macros, which asserts for equality, stopping execution if the arguments are not equal. You may also use `EXPECT_EQ` if you want to continue execution regardless.
  4. You can set GoogleTest up by calling `testing::InitGoogleTest`.
  5. Then, start GoogleTest up by invoking `RUN_ALL_TESTS`.

# Compilation

Depending on your setup and how you've installed the library, you may be able to compile by typing:
   
    g++ -o calc_test calc_test.cpp calc.cpp -lgtest -lpthread

Note that you must link the code against the GoogleTest ( _gtest_ ) and the
POSIX Thread ( _pthread_ ) libs.

# Running

To run the test suite, just type:   
    
    ./calc_test

Thus, the test runner will run each test-suite and print a summary similar to this:

![Summary](/assets/img/2018-02-19-introduction-to-google-cpp-unit-testing_0.png)

# Conclusion

During software development, life-cycle is important to write tests to
check for correctness, thereby paving the way for improvements later down the road by boosting developer's confidence when refactoring, etc.

In this context, tools for automated testing are critical to helping us out in the process. They should be integrated
into Continuous Integration (CI) pipelines to be executed automatically upon pushes.

# References

[1] <https://github.com/google/googletest#Assertions>  
[2] <https://www.eriksmistad.no/getting-started-with-google-test-on-ubuntu/>

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
