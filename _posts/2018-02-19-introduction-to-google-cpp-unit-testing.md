---
layout:	"post"
title:	"Introduction to Google C++ Unit Testing"
---

* * *

Hello,

Today, I'm going to introduce the Google Testing Framework for C++,
abbreviated as GoogleTest.

### Installation

To install GoogleTest on Debian, I've used the _apt_ command:

    
    
    sudo apt install libgtest-dev

This command will only install the sources into your _src_ directory (for
example: _/usr/src/gtest_ ), now you have to compile them with the tools:
_cmake_ and _make_ :

    
    
    cd /usr/src/gtest sudo && cmake CMakeLists.txt && sudo make

Then, copy the generated binaries into your lib directory (for example:
_/usr/lib_ ):

    
    
    sudo cp libgtest.a libgtest_main.a /usr/lib

### Example

This example will be a simplified calculator, named **calc** , for integral
numbers computations with just two supported operations: addition and
subtraction.

NOTE: I've ignored the code comments to simplify the understanding, but
remember to comment your code whenever necessary.

The header file _calc.hpp_ :

The implementation file _calc.cpp_ :

And finally, the test file _calc_test.cpp_ :

There are some new stuff from GoogleTest here:

  1. You must include the header file _gtest/gtest.h_ (look at _/usr/include/gtest_ for more information)
  2. The _TEST_ macro is responsible for the creation of the test cases, where the first parameter is the test suite name and the second parameter is the specific test case name, which will be visible in the display information during execution
  3. The _ASSERT_EQ_ , member of _ASSERT_*_ group of macros, defines the equality test for GoogleTest and, based on this information, the framework verifies the success or failure of the tests and logs these information, aborting the test case when the test failed. You can use _EXPECT_EQ_ if you want to continue the test case execution in the presence of a failure
  4. You can prepare the GoogleTest by calling _testing::InitGoogleTest_ and passing a pointer to _argc_ and the _argv_ array (which is a pointer too)
  5. Then, you start GoogleTest by invoking _RUN_ALL_TESTS_

### Compilation

To compile the example, type the following command:

    
    
    g++ -o calc_test calc_test.cpp calc.cpp -lgtest -lpthread

Note that you must link the code against the GoogleTest ( _gtest_ ) and the
POSIX Thread ( _pthread_ ) libs.

### Running

Finally, to run the test suite, just type:

    
    
    ./calc_test

Thus, the test runner will run each test suite and output a summary similar to
this:

![](/assets/img/2018-02-19-introduction-to-google-cpp-unit-testing_0.png)

### Conclusion

During software development life cycle is very important to write test code to
verify the correctness of production code, paving the way to improve the code
quality, developer's confidence when refactoring etc. In this context, tools
for automatic unit testing can help in the process and they can be integrated
with a Continuous Integration (CI) server to be executed automatically for
each commit.

In a future post, I intend to talk more about unit testing and the software
development methodology denominated Test Driven Development (TDD), which
changes the "conventional" process from: Code -> Test, to: Test -> Code.

### References

[1] <https://github.com/google/googletest#Assertions>  
[2] <https://www.eriksmistad.no/getting-started-with-google-test-on-ubuntu/>

