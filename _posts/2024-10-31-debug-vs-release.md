---
layout: post
title:  Debug vs Release
date:   2024-10-31 23:19:00 -0600
tags:
- C++
- Windows
---
I exclusively used Linux for programming for approximately as long as I've been programming. Now at a job where I write software for Windows users, I'm learning the differences between these OSes as programming environments, and the differences between their respective commonly used C++ compilers.

Work uses Visual Studio and the MSVC compiler. I only recently learned that, in MSVC, "Debug" and "Release" are separate DLLs. While I imagine this is obvious to most Windows-using C++ developers, it surprised me because my mental model of "Debug" configuration was simply "the same as Release but with Debug symbols and without compiler optimizations". This reflects how I came to build in "Debug" in the first place. At work, I was taught to build with "Debug symbols in Release", so I was already debugging in "Release" configuration. I was stepping through some code with the debugger in Visual Studio when it suddenly skipped over all the lines of code I had wanted to debug. That code had been optimized away by the compiler. My coworker suggested making a "Debug" build to avoid this. Now I believe he wanted to give me a guaranteed solution instead of getting into the nuances; since then, I have learned the common workflow (at least at my workplace) is to toggle off optimization for specific projects and recompile them as needed when debugging. Given the ubiquity of "Debug symbols in Release" in my limited experience, in my mind, the "Debug" configuration was simply a quick way of telling MSVC to disable compiler optimization for all projects.

However, I noticed that, sometimes, tests that passed in a "Release" build would fail in a "Debug" build. Compiler optimization should not affect the outcome of running some code! What could explain this difference?

The other major distinction between "Release" and "Debug" configurations is that "Debug" does more automatic initialization -- setting memory to zero or to a recognizable "null" value that denotes it as not being initialized by the code being debugged -- and checks for out-of-bounds memory access. This could come in the form of a pointer referencing a deleted object or an iterating past the end of an array. Perhaps in a Release build, the object hadn't yet been deleted (just marked for deletion) or the array happened to be co-located in memory with some other valid data. Perhaps this behavior is fairly consistent when running the test on the same computer with the same data. Perhaps this is why C++ is considered "dangerous" and "memory unsafe". Perhaps it's good to test with a "Debug" build when possible -- if it's not too slow to run the tests.

For whatever reason, I've found it difficult to locate this information with Google searches. The top hits and the AI summary are only about debug symbols and compiler optimizations, or Stack Overflow posts saying "There is no difference between Debug and Release except what your company says it is." I assume this is the case for Linux devs using gcc or clang, which don't have dedicated "Debug" configurations with separate linked libraries. For posterity / my own reference, Microsoft lists these extra memory-checking "Debug" configuration features [here](https://devblogs.microsoft.com/cppblog/security-features-in-microsoft-visual-c/#when-you’re-testing-your-code). Quoted in case the web page changes or disappears:

> The compiler toolset offers many options that help you when you’re testing your code. Most of these switches aren’t intended to be shipped with your final, retail builds of your program. They’re turned on for debug builds—either by default or opt-in—so that you can find more bugs during your testing.

> CRT Debug Heap

> The CRT Debug Heap is enabled when you compile your program in debug (non-release) mode. It finds common heap memory errors, including buffer overruns and leaks. The CRT Debug Heap will assert when it encounters any heap memory errors as you test your code. [Various debug routines](https://docs.microsoft.com/en-us/cpp/c-runtime-library/debug-routines) are enabled when you define the `_DEBUG` flag.

> Runtime Checks

> The CRT provides [Runtime Checks](https://docs.microsoft.com/en-us/cpp/build/reference/rtc-run-time-error-checks) enabled through use of the /RTC switch. These checks find real logic errors in your program such as data loss, initialization issues, and stack frame checking. Runtime Checks are only intended for when you are active testing your debug builds and are incompatible with optimizations. Because of these limitations they are off by default.

> Checked Iterators

> Checked iterators help ensure that your code doesn’t accidentally overwrite the bounds of iterable containers in your code. They can be used both in [debug code](https://docs.microsoft.com/en-us/cpp/standard-library/debug-iterator-support) (as debug iterators) and in [release code](https://docs.microsoft.com/en-us/cpp/standard-library/checked-iterators) (as checked iterators.)

