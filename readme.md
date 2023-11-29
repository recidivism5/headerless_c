# Headerless C
So you want to code C across multiple files without maintaining two copies of each symbol signature (declaration+definition)?

Put the following code at the top of all your .c files:
```c
#pragma once
#ifdef __INTELLISENSE__
#undef INCLUDED
#endif
#ifdef INCLUDED
#define INCLUDED 1
#else
#define INCLUDED 0
#endif
#pragma push_macro("INCLUDED")

/*
put all your includes here.
for example:
#include "test.c"
#include <stdio.h>
*/

#pragma pop_macro("INCLUDED")
```

Then for each function do this:
```c
int randint(int n)
#if INCLUDED == 0
{
    if ((n - 1) == RAND_MAX){
        return rand();
    } else {
        int end = n * (RAND_MAX / n);
        int r;
        while ((r = rand()) >= end);
        return r % n;
    }
}
#else
;
#endif
```

### Reasoning:
This conditional compilation requires either a stack or counter for include depth.
If you try to do it without one you will run into this problem:
```c
#define INCLUDE
#include "test1.c" //the expansion of this contains "#undef INCLUDE", which causes test2.c to be included with implementations
#include "test2.c"
#undef INCLUDE
```

Modern versions of GCC support a macro called `__INCLUDE_LEVEL__` that has the value of the include level, i.e. 0 when not included, nonzero when included. This would solve our problem but it is not available on MSVC and apparently is not very common in general. So instead we use the much more ubiquitous `#pragma push_macro(...)` macro to give us an include level stack. Of course, our `INCLUDED` value is only ever 0 or 1 with this solution but that's not an issue for our use case. This solution supports msvc, clang, gcc.

`#pragma once` is only needed if this file is included multiple times in one translation unit. But it's 2023 so you should probably just use it regardless.

Intellisense tries to be smart and grey out implementations based on files including this file. So we do an ifdef to stop that.

This was inspired by https://github.com/milgra/headerlessc

### Further notes:
Visual Studio doesn't understand that our .c files are also effectively headers. So when we change a .c file, any .c file relying on that file will not automatically be recompiled. So we have to rebuild our project each time. Right clicking on our project -> Project Only -> Rebuild Only \[Project Name\] solves this problem, but is annoying to do every time we want to debug our project. Instead we want to just click the Debug play button at the top and have everything work. The only solution I've found is to go to Project Properties -> Build Events -> Pre-Build Event and add a command like `cd your_project_name_here.dir && del /s *.obj` to delete every object file in your main project. This works perfectly.
