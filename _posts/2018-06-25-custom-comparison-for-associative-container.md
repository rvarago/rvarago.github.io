---
layout: "post"
title:  "Custom Comparison for Associative Container"
tags:   stl c++
---

> For STL associative containers, the type of keys stored must be comparable. Although in some cases you might want to supply an alternative comparison function.

* * *

To use the associative containers provided by the Standard Template Library (STL), the type of
keys stored must be somehow comparable.

For ordered containers (e.g `std::set<T>`), the comparison of elements of type `T` is by equivalence,
which, roughly speaking, means overloading `operator<` for `T`. And for unordered containers (e.g `std::unordered_set<T>`) the comparison is by equality, which in, again roughly speaking, means overloading `operator==`.

Nevertheless, for some instances of a container, we may want to supply an alternative comparison function that makes more sense for that single container instance.

For example, given a type `Person` that would normally sorted by `Person::id`, for some `std::set <Person>` called `names` we may want to compare the elements by `Person::name`, instead of `Person::id`. How do you "override" the default comparison function of `Person` for this particular instance of `std::set <Person>`?

## The Complete Declaration for Associative Containers

To exemplify the concepts, I'll pick up `std::set` as a representative of the
STL's associative containers, albeit differences between it and others, aren't significant for our goal, which is how to create an associative container with a different comparison function than the one defined by the type of element held inside the container?

According to [cppreference](https://en.cppreference.com/w/cpp/container/set), the complete declaration of `std::set` is:

```cpp
template<   
  class Key,   
  class Compare = std::less<Key>,   
  class Alloc = std::allocator<Key>  
> class set;
```

Where:

 *  `Key`is the type of elements held in the container.
 *  `Alloc` is the memory allocator, with the default being `std::allocator<Key>`.

And the most important piece for us:

* Compare is the type of comparison used by the container, which defaults to `std::less<Key>`.

Now, what's `std::less<Key>`?

`std::less<Key>` is a function object that expresses the "less-than" operation on objects of type `Key`. And it does so by calling `operator<` overloaded by the type `Key` to check the relative order of two objects `A` and `B` of type `Key` like `A < B`.

Given the declaration of `std::set`, we see that the way to provide our desired behaviour is to supply a different type of comparison function
object for the template parameter: `Compare`. Alright? Let's do it then!

Our type `Person` might look like this:

<script src="https://gist.github.com/rvarago/e018a7dae52619894458b8e56de0f385.js"></script>

The use of `Person` inside `std::set` and comparing by `Person::name`, instead of `Person::id` can be done by:

<script src="https://gist.github.com/rvarago/65bea0e13527830724127ecc2e1f91a0.js"></script>

Firstly, we created a variable `peopleById` that uses the internal (default) comparison function for types `Person`: `Person::id()`. Thus, every element inside the container will be sorted based on its `id`.

Then, at (1) we created `peopleName` that relies on the function object `PersonCompareByName` as the argument for the template parameter `Compare`, establishing the comparison by `Person::name()`.

At (2) we achieved the same behaviour of (1), but employing an alternative syntax that uses a lambda expression `personComparator` and supplied its type to the template parameter with `decltype`.

Finally, the output of the program should be similar to:

![Output](/assets/img/2018-06-25-custom-comparison-for-associative-container_0.png)

# Conclusion

Sometimes we may want to supply alternative ways to sort elements of associative containers, other than the one defined by the key part of the
element (i.e. `T::operator<`), but the custom behaviour may be wanted for only a particular instance of the container. This text has shown a way to achieve the intended behaviour, by providing two different syntaxes to achieve the same result.

STL has a vast collection of algorithms and containers that simplify our daily programming tasks in C++, with highly flexible and optimized solutions. Therefore, I encourage you to search for more information and whenever possible, favour STL components in your projects rather than home-made solutions.

# References

[1] [http://www.cplusplus.com/reference/set/set/](http://www.cplusplus.com/reference/set/set/)

[2] [http://www.cplusplus.com/reference/functional/less/](http://www.cplusplus.com/reference/functional/less/)

[3] Effective STL. Scott Meyers.

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
