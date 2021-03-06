================================================================================
                            Cleanup C++/C headers
================================================================================

This simple script helps in removing unnecessary includes from C/C++ files.

A huge code base or legacy code usually means that implementation files are
full of includes pilled up over years. Likewise, creating a new project by
forking an old one ends up with tons of leftovers.


How it works?
-----------------------------------------------------------

The only thing you need is **the full command** that creates an object file.

The script systematically comments out one include file at once and recompiles
source. When program/object file still compiles, then the commented out include
is considered unneeded.

**Caveat 1** (noticed by my colleague Leszek): such a mechanical way of removing
includes may lead to creating indirect dependencies. Some symbols required by
implementation might be provided (accidentally) by includes present in headers
files. When one remove include from the implementation file, then later changes
to the header file might break the compilation.

**Caveat 2**: the script doesn't interpret any other preprocessor directive
than ``#include``. Conditional includes enabled by ``ifdefs`` will be removed
unconditionally.


How to use it?
-----------------------------------------------------------

Here's an example from my toy project https://github.com/WojciechMula/avx512popcnt-superoptimizer

The head of the main program::

    $ head -n 15 avx512popcnt.cpp
    #include <cstdint>
    #include <cstdlib>
    #include <cstdio>
    #include <cassert>
    #include <vector>
    #include <memory>
    #include <random>
    #include <algorithm>
    #include <bitset>

    #include <sys/types.h>
    #include <unistd.h>

    #include "binary.cpp"

When run ``make``, we capture the command which builds the program::

    $ make avx512popcnt
    g++ -Wall -Wextra -pedantic -std=c++14 -O3 avx512popcnt.cpp -o lineavx512popcnt

Now the script comes::

    $ python cleanup.py g++ -Wall -Wextra -pedantic -std=c++14 -O3 avx512popcnt.cpp -o avx512popcnt
    Checking compilation of avx512popcnt.cpp... OK
    Removing cstdint (1/12)... OK
    Removing cstdlib (2/12)... OK
    Removing cstdio (3/12)... OK
    Removing cassert (4/12)... not possible
    Removing vector (5/12)... OK
    Removing memory (6/12)... not possible
    Removing random (7/12)... not possible
    Removing algorithm (8/12)... OK
    Removing bitset (9/12)... OK
    Removing sys/types.h (10/12)... OK
    Removing unistd.h (11/12)... not possible
    Removing binary.cpp (12/12)... not possible
    avx512popcnt.cpp: not required cstdint, cstdlib, cstdio, vector, algorithm, bitset, sys/types.h
    avx512popcnt.cpp was updated

It turned out that eight includes weren't needed at all.


Configuration
-----------------------------------------------------------

You can control behaviour of the script via a config file. The config
file must be located either in ``~/.config/cleanup-headers/config.ini``
or its path must be provided by the environment variable
``CLEANUP_HEADERS_CONFIG``. Please refer to the sample ``config.ini``
for more details.
