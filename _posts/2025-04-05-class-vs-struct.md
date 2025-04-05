---
layout: post
title:  class vs. struct
date:   2025-04-05 11:23:00 -0600
tags:
- C++
- initialization
---
"What is the difference between a class and a struct?" might be a common C++ interview question. I was asked this during the final boss interview for my current job, and I failed to give the correct answer. Googling it today, the AI Overview says:

"The difference between struct and class in C++ lies primarily in their default access specifiers and inheritance defaults.

Default Access Specifier:
* **struct**: Members are public by default. This means they can be accessed directly from outside the struct.
* **class**: Members are private by default. Access to them is controlled through member functions (getters and setters).

Inheritance Default:
* **struct**: When inheriting, the default access specifier is public.
* **class**: When inheriting, the default access specifier is private.

While these are key differences, in practice, struct and class can be used interchangeably in many situations. Both can contain data members, member functions, constructors, destructors, and can use inheritance and polymorphism."

So, stuff in a struct is public, except you can choose to make it private, and stuff in a class is private, but you can choose to make it public, and the answer plainly states that they're basically interchangeable. Then why ask the question in an interview? Is it so important to know the trivia, "struct is to public as class is to private", especially when that's merely a default? I give the final boss more credit than that; he wouldn't ask a mere trivia quesiton. Rather, he wanted to get at the deeper implications, and why there might be a reason to choose one over the other. Now, a full year into working with this code base, I can go beyond mimicking the pattern we use and explain why it was chosen.

At work, our pattern is to use a struct instead of a class to indicate to other developers that **aggregate initialization** can be used. That means the members of the aggregate (which means: an array, class, or struct that allows direct access to its members) are initialized directly, without a(n explicit) constructor. The syntax is a brace-enclosed ordered list like:

```
struct Data
{
    int count;
    double mean;
    string name;
}

Data d = {10, 2.718, "z"};
```

If the number of initializers is less than the number of bracketed items, the remainder are zero-initialized:

```
Data d = {10, 2.718};  // name is value-initialized to an empty string.
```

C++20 allows this dot notation called "designated initializers" to specify the members explicitly:

```
Data d = {.count = 10, .mean = 2.718};
```

Aggregate initialization is possible when there are:
* No private or protected non-static data members.
* No explicitly declared constructors (though the explicitly declared default constructor is allowed).
* No virtual functions.
* No virtual, private, or protected base classes or inherited members.

The first bullet point is key on why structs are preferred. By default, their members are public; a developer would have to go out of their way to change this behavior and make aggregate initialization not possible. Classes are the other way around, so we use them when more complicated, explicitly defined constructors are required. What forces a constructor to be "more complicated"? By this I mean, simple direct assignment to members is not possible; some kind of logic is required before the assignment can be made. Specifically, this refers to **invariants**, meaning, non-independence of data members or constraints that are placed on data members. For example, a value assigned to one data member restricts the possible range of values that can be assigned to another data member.

In short: If a class has no invariant, use a struct instead.
But also, it's kind of just convention; you could do exactly the same things with a class by adding "public:" before the members.

This video makes the same recommendation and helped me understand why we follow this pattern at work:

[Writing C++ to Be Read](https://www.youtube.com/watch?v=ABnf8NV6yEo)

