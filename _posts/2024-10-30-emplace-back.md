---
layout: post
title:  emplace_back() and other implicit constructors
date:   2024-10-30 19:56:00 -0600
tags:
- C++
- constructors
---

This is a lesson I will never forget, and the lesson that inspired me to start blogging.

It's also super obvious in retrospect, but was equally super bewildering at the time.

I was struggling with a random bug in the code base at work that crashed my simulations after unpredictable amounts of runtime. The randomness was not unexpected -- this was in a Monte Carlo code! -- but it presented obvious challenges. This was occurring on a computer without, at the time, access to an IDE or any sort of debugger beyond the error messages I could insert into thrown exceptions (like "printf debugging" but even worse). I could see exactly where the exception was thrown: within the constructor of an object that was being passed some invalid data, specifically a zero. The constructor contained a check for this, and threw the exception with a specific, uniquely attributable error message; it was easy to track down, and to be certain it was not coming from anywhere else in the code base. However, I could not pinpoint from where the constructor was being called in a "random" way.

Being a C++ noob, my only idea was to search the code base for the class name followed by an open parenthesis -- the obvious way in which a constructor is called, explicitly -- or for declarations, where the constructor is called implicitly but obviously.

	// Defining the constructor inside the class
	MyClass {
	    MyClass(parameter) { ... }
	}

	// Defining the constructor outside of the class
	MyClass {
	    MyClass(arg);
	}
	MyClass::MyClass(parameter) { ... } 

	// Declaration, calling the constructor implicitly but obviously
	MyClass instanceOfMyClass(arg);

	// Calling the constructor explicitly
	returnedValue = aFunctionThatTakesAMyClassObject( MyClass(arg) );

(The code conventions of this code base justifiably prohibit the use of "new", so a declaration would not involve calling the constructor explicitly.)

This enabled me to find one place in the code where the constructor was called with a randomly-generated number that could have been zero -- though the chances of that occurring seemed infinitesimal. Much can be said about the true randomness of procedurally generated pseudo-random numbers, and I have not performed an exhaustive investigation of whether the value zero showed up more than should be expected statistically with the RNG used in this code base. I added a check for zero and went on my way, though with trepidation, not at all convinced I had fixed the problem.

Sure enough, the next rounds of simulations crashed randomly with the same error. Lacking any modern tools or sufficient language knowledge to debug the issue, it felt almost hopeless. I did my best to try to narrow down the conditions under which the error occurred (was it due to a particular starting condition in the simulation setup?), and to add more logging, but I couldn't figure it out.

What I should have asked myself is: How else is a constructor called?

Out of beginner naivete, I had overlooked the majority of ways to call a constructor, which are *implicit*.

Besides initialization, as mentioned above, the constructor is called implicitly when returning an object from a function.

	MyClass createObject() {
	    return {arg};  // calls MyClass(arg)
	}

But what really got me -- as you have undoubtedly surmised from the title of this post -- is that the constructor can be invoked implicitly when inserting into standard containers, using methods like "emplace".

	std::vector<MyClass> myVec;
	myVec.emplace_back(arg);  // calls MyClass(arg)

	std::unordered_map<int, MyClass> myMap;
	myMap.emplace(1, arg);  // calls MyClass(arg)

I'm deliberately overlooking ways in which a copy constructor can be called implicitly, since in my case, the object could not be created in the first place with the invalid value.

As it turned out, I finally got an IDE with debugger onto the computer and solved the problem within 20 minutes. Letting the simulation run in the debugger gave me a useful stack trace (somehow the stack trace I got from logging attempts was different and not helpful? I'm still learning...) that pinpointed a call to `emplace_back()` on a vector of elements of the class in question. It was in a loop creating semi-random intervals, with insufficient checking that the start and end times were sufficiently different from one another.
