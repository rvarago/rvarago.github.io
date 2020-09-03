---
layout:	"post"
title:	"Custom Comparison for Associative Container"
---

To use the STL associative containers, the type of keys stored must be
comparable. But for some instances of the containers, you might want to supply
an alternative comparison function.

* * *

To use the Standard Template Library (STL) associative containers, the type of
keys stored must be comparable.

For sorted containers (e.g _std::set_ ), the comparison is by equivalence,
which **roughly** means overloading the _operator <_, __ and for unsorted
containers (e.g _std::unordered_set_ ) the comparison is by equality, which in
turn (again, **roughly** ) means overloading the _operator==_.

But for some instances of a container with a predefined comparison function
for the stored type, you may want to supply an alternative comparison function
that makes more sense for the container instance's purpose.

For example, given a type _Person_ that are normally sorted by _Person::id_ ,
for some _std::set <Person> _called _names_ you may want to compare the
elements by _Person::name_ instead of _Person::id_. How do you  "override" the
default comparison function for _Person_ to achieve the desired behavior for
this particular instance of _std::set <Person>_?

#### The Complete Declaration for Associative Containers

To exemplify the concepts, I'll consider _std::set_ as a representative of the
STL 's associative containers, the differences between it and the others
aren't significant for our purpose: how to create an associative container
with a different comparison function from the one defined by the type of
element held inside the container?

The complete declaration for _std::set_ is:

    
    
    template   
     <   
    class T,   
    class Compare = less<T>,   
    class Alloc = allocator<T>  
    >   
    class set;

Where:

  *  _T_ is the type of elements that will be held
  *  _Alloc_ is the memory allocator, with the default being _allocator <T>_

And the most important aspect for this article:

> Compare is the type of the comparison used, with the default being
_std::less <T>_

But, what's _std::less <T>_?

 _std::less <T> _is a function object that expresses the "less-than" semantic
for objects of type _T_. Its behavior is to call the _operator <_ overloaded
by the type _T_ to check the relative order of two objects _A_ and _B_ of type
_T_.

Given the declaration for _std::set_ , we can see that the way for providing
our desired behavior is to supply a different type of comparison function
object for the template parameter: _Compare_. Alright? Let 's do it then!

Our type _Person_ looks like this:

And the use of _Person_ inside _std::set_ and comparing by _Person::name_
instead of _Person::id_ can be achieved by:

Firstly, we'd created the _peopleById_ that uses the internal (default)
comparison function for the type _Person_ : By _Person::id()_. Thus, every
element inside the container will be sorted based on its id.

Then, at (1) we created _peopleName_ that relies on the function object
_PersonCompareByName_ as the argument for the template parameter _Compare,_
establishing the comparison by _Person::name()_.

At (2) we achieved the same behavior of (1) but employing an alternative
syntax that uses the lambda _personComparator_ and supplied the template
parameter with _decltype_.

Finally, the output of the program will be:

![](/assets/img/2018-06-25-custom-comparison-for-associative-container_0.png)

#### Conclusion

Sometimes you may want to supply an alternative way to sort elements of
associative containers, other than the one defined by the key part of the
element (e.g. _T::operator <_), but the custom behavior may be wanted for just
a particular instance of the container. This article has covered a manner to
achieve this goal, providing two alternative syntaxes for expressing the
intention.

STL has a vast collection of containers and algorithms that simplify our daily
C++'s programming tasks in a highly flexible and algorithmically optimized
manner. Therefore, I really encourage you to search for more information and,
whether possible, favor STL's components in your projects rather than home-
made solutions.

#### References

[1] <http://www.cplusplus.com/reference/set/set/>

[2] <http://www.cplusplus.com/reference/functional/less/>

[3] Effective STL. Scott Meyers.

