---
layout: post
title:  initialization
date:   2025-04-05 15:48:00 -0600
tags:
- C++
- initialization
---
My previous post about classes vs. structs and the implication for aggregate initialization led me down a rabbit hole about initialization more generally. I found a good YouTube video on this topic and want to record my notes here for my own benefit.

# Initialization vs Assignment
Initialization declares the object and gives it an initial state. Both of the below lines are examples of initialization.
```c++
Data d1(1234);
Data d2 = d1;
```

If the object already exists, e.g.
```c++
Data d3;  // initialization
d3 = d1;  // assignment
```
(Note we are not defining the type of d3 in the assignment.)

This is obvious; what's the point?
- Assignment invokes `operator=` rather than the copy constructor.
- The distinction exists because assignment replaces the value of an existing object; it may need to clean up the object's previous state, e.g. delete allocated memory.

# Copy Initialization 
Used when the initialization values come after an equals sign ( = ). This is the traditional C way of initializing, though C itself does not name the forms of initialization like C++ does.
Examples:
```c++
int x = 1;
int *px = &x;
```

In C++, copy initialization uses the copy constructor. But copy initialization is not the only initialization that can use the copy constructor.
```c++
Data d1 = {1234, 0.1};
Data d2 = d1;  // copy initialization using the copy constructor
Data d3(d1);   // direct initialization using the copy constructor
```

In addition to the equals sign, copy initialization occurs when:
- passing or returning an object by value
- throwing an exception
- catching an exception by value

# Aggregate Initialization
A form of copy initialization applied to classes or structs that have only public data members (and have no virtual methods, etc.).
Examples:
```c++
struct widget {
    int id;
    double price;
}

widget w1 = {1234, 6.5};  // aggregate initialization
widget w2 = w1;           // copy initialization
```
- "Neither C nor C++ lets us copy initialize an array" -- an array is considered an aggregate, but it cannot be aggregate initialized.
- As of C++20, aggregates can be initialized with parentheses as well as with braces.

# Zero Initialization
C++ follows the zero initialization rules from C:
- Scalars are initialized as if with the value 0.
- For aggregates, every element is initialized as if with the value 0.
    - If the brace-enclosed list in aggregate initialization doesn't cover every element of the aggregate, the remaining elements are zero initialized. Example: `widget w1 = {1234};` is equivalent to `widget w1 = {1234, 0.0}`, from the aggregate initialization example above.
- Pointers are initialized to NULL / `nullptr`.
- Otherwise, the object is uninitialized, a.k.a. has indeterminate value, and accessing it produces undefined behavior (UB).

An object without an explitit initializer is zero initialized if it has static or thread storage duration. That could be:
- Declared `static`
- Declared `thread_local`
- **At namespace scope** 

At work, we require everything to be in a namespace, so the last bullet point is key.

# C++ vs C
C++98/03 introduced more initialization rules beyond the C rules, having to do with C++ notions of class types, constructors, and object lifetimes.

C makes no distinction between when memory is allocated for an object and the start of that object's lifetime. In C, lifetime == storage duration.

C aggregates are simple. They do not allow consideration of **invariants**, i.e. constraints between data members. For example,
```c++
struct demo_str
{
    std::size_t str_length;
    char* the_string;
}
```
The invariant here is the relationship between `str_length` and `the_string`; `str_length` is supposed to indicate the length of `the_string`. C relies on the programmer to respect the invariant. There is no compiler check in C to disallow instantiations like `demo_str ds = {5, NULL};` (an empty string of length 5?). **C++ constructors arose to enforce class invariants.** This complexity is why aggregate initialization isn't supported with explicitly defined constructors.

# "Which initialization do you mean?"
Even if an object is copy initialized, its members can be direct initialized.
```c++
// Copy constructor
demo_str::demo_str(demo_str const &other):
    str_length(other.str_length),         // direct initialization
    the_string(new char[str_length + 1])  // direct initialization
{
    std::strcpy(the_string, other.the_string);
}
```

With aggregate initialization, each individual field is copy initialized.

Why care about this distinction? **It affects which coversions are permitted.**

A constructor that is not declared with the keyword `explicit` is a coverting constructor. It provides an implicit, user-defined conversion from the constructor's parameter types to the class type. Example:
```c++
void f(demo_str ds);  // pass by value
f("McCoy");           // copy initialization
```
The call to `f` here is allowed because copy initialization enables the conversion of the (string decayed to a) pointer to a `demo_str`, even though we did not explicitly pass a `demo_str` to `f`.

## explicit Constructors

If the `demo_str` constructor were declared as `explicit`, conversions would no longer be allowed and the call to `f` above would fail.

# Default Initialization

Initialization using the default constructor, called by omitting the parenthesis: `demo_str ds;` as opposed to `demo_str ds();`

In C++, just like in C, a scalar object with no initializer is left uninitialized.
```c++
int x;  // uninitialized
```

However, C++ still refers to `x` as default initialized to describe when the lifetime of `x` begins.

Sometimes, this do-nothing form of initialization is called *vacuous initialization*.

# Value Initialization

## "Most Vexing Parse"
The compiler will always interpret a declaration with parentheses as a function (instead of a variable) if it is valid to do so. This comes from how C was designed and is preserved in C++ for backwards compatibility. This can lead to unexpected parsing, e.g. you think you should be able to declare a vector as
```c++
vector<int> v1(istream_iterator<int>(cin), istream_iterator<int>());
```
but it is interpreted as a function. (The complier even ignores the parentheses after the `istream_iterator`s as superfluous.)

But we can initialize objects using parentheses in other contexts. This is valid since C++03:
```c++
int x = int();              // x is 0
demo_str ds = demo_str ();  // calls default constructor then copy constructor
```
(In C++03, this required an accessible -- i.e. public -- copy constructor, although this is no longer necessary as of C++17 rules about *copy elision*.)

## Method of Value Initialization
- If object has a user-provided default constructor, call it.
- If object is an array, value initialize all of its elements.
- If object is a scalar, zero initialize it.
- Otherwise, the initialization is valid only if the object has no user-provided constructors.

For each data member of the object:
1. Zero initialize it.
2. Then, default initialize it (i.e. call the default constructor).

# Braces as Uniform Value Initialization Syntax

Modern C++ introduced brace initialization (**direct list initialization**) as a uniform initialization method that can be used by all types. C++03 lacked this, and coupled with the "vexing parse" parentheses problem inherent to supporting legacy C, it meant different types had to be initialized in different ways, placing restrictions on templates / making templates less helpful.

## Copy List Initialization

Using braces with = is a form of copy initialization called *copy list initialization*.

Same thing:
```c++
int x = 5;
int x = {5};
```

Also the same thing:
```
// Both of these call demo_str(char const *)
demo_str ds1 = "Picard";
demo_str ds2 = {"Picard"};
```

## Copy List Initialization with explicit Constructors

If the copy constructor of `demo_str` were declared explicit, then direct list initialization would permit conversions from `char const *` to `demo_str` (assuming the direct constructor was not also declared explicit) but copy list initialization would not.

```c++
class C
{
public:
    C(int x, demo_str const &str);
}

C c1{10, "Riker"};     // Allowed
C c2 = {10, "Riker"};  // Error, requires implicit conversion
```

## Narrowing Conversions

Braced initialization prohibits *narrowing coversions*, i.e. implicitly converting to a smaller type that could result in loss of data. It is only permitted if the initializer is a constant expression and the complier can verify that there will be no loss of data.
```c++
int i = 12;
char itoc1{i};   // Error: no narrowing allowed
float itof1{i};  // Error: no narrowing allowed

constexpr int a = 12;
char itoc2{a};   // Allowed if 12 fits into a char
float itof2{a};  // Allowed if 12 fits into a float
```

## std::initializer\_list Constructors

Technically it is possible to write a constructor specific to initializer (bracketed) lists, like
```c++
class demo_str
{
public:
    demo_str();
    demo_str(std::initializer_list<char> init);
}
```
but why do this. In general, *don't do this*.

- If a class has both a default constructor and a `std::initializer_list` constructor, empty braces invoke the default constructor.
- If a class has a `std::initializer_list` constructor and doesn't have a default constructor, empty braces are interpreted as an empty `std::initializer_list`.

`std::initializer_list` supports iterators, just like the STL container types. (`std::vector` is a template that has a `std::initializer_list` constructor.)

Unfortunately, if a class type has a `std::initializer_list` constructor, it is possible for these two definitions to invoke different constructors:
```c++
InitializerListClass c1(1, 2);
InitializerListClass c2{1, 2};
```
This is most common when InitializerListClass is a class template type, such as `std::vector<T>`. The confusion usually occurs when the `std::initializer_list` holds a type that closely resembles arguments to other constructors. In the example below, 12 can be interpreted as either `std::vector<int>::size_type` or `std::initializer_list<int>`.
```c++
std::vector<int> v1( 12 );  // std::vector<int>::size_type
std::vector<int> v2{ 12 };  // std::initializer_list<int>
```

Because of this, you have to be careful about using brace initialization in templates -- even though universal syntax for templates was part of the reason for its creation.


Source:

[Back to Basics: Initialization in C++ - Ben Saks - CppCon 2023](https://www.youtube.com/watch?v=_23qmZtDBxg&t=298s)
