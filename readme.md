# Headerless C
So you want to code C across multiple files without maintaining two copies of each symbol signature (declaration+definition)?

Put the following code at the top of all your .c files:
```c
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
Modern versions of GCC support a macro called `__INCLUDE_LEVEL__` that has the value of the include level, i.e. 0 when not included, nonzero when included. This would solve our problem but it is not available on MSVC and apparently is not very common in general. So instead we use the much more ubiquitous `#pragma push_macro(...)` macro to give us a stack. Of course, our `INCLUDED` value is only ever 0 or 1 with this solution but that's not really an issue. This solution supports msvc, clang, gcc.