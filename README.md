wlibc
=====

A BSD libc for Windows that works!

Executive summary
-----------------

On Windows, one of the most common C library implementations is Visual Studio's C Run-Time (CRT). Unfortunately, it's far from source-compatible with glibc or Darwin/FreeBSD libc, and includes an incredible quantity of "unusual" implementation decisions that make it really very hard to port *nix applications to Windows, if they include many calls to ISO C functions. Replacing POSIX functions with Win32 ones is fair enough, but standard C is supposed to be a vendor-neutral language!

This project aims to offer a drop-in replacement for the Visual Studio CRT that's as compatible as possible with a Unix libc, minimizing changes to low level code. Portable applications will therefore not need to reimplement basic stuff that's broken in the Visual Studio CRT. In particular, only UTF-8 shall be supported ("C" is an alias for the "C.UTF-8" locale), and all narrow functions (such as `fopen`) assume the argument is UTF-8 encoded.

Detailed list of differences
----------------------------

### Things completely broken in the CRT

These are show-stoppers, deficiencies in the CRT that render the relevant functions completely useless.

1. Neither `strftime` nor `wcsftime` functions. The narrow version can't be passed unicode characters, and the wide version purposefully mangles characters that aren't in the current single-byte codepage. See [this StackOverflow question](http://stackoverflow.com/questions/20971039/how-do-i-get-wcsftime-to-work-in-visual-studio-crt) for details. A functioning replacement must be provided.
2. `printf` can't be used to print out Unicode. If your application calls `printf` or `fprintf` anywhere, it won't work if you try to compile it with the Visual Studio CRT. If you try to use `wprintf`, you'll have a bit more success, but there are several bugs in the handling of UTF-8, and the provided `_O_U8TEXT` and `_O_U16TEXT` modes are hard to use (to put it generously). [Alf Steinbach's excellent article](http://alfps.wordpress.com/2011/12/08/unicode-part-2-utf-8-stream-mode/) explains the issues more thoroughly, but the basic take-home is that `printf` is basically off the cards with the CRT unless you do some insane backflips. Replacing the whole thing is easier.
3. `setlocale` is generally useless because you can't coerce it into producing UTF-8 output (not that it matters much, since the relevant locale-aware functions like `wcsftime` randomly limit their output to that of a single-byte codepage anyway).

### Completely gratuitous breaking of source compatibility

These issues can be worked around, but they just make porting to Windows an incredibly ugly process. Why pollute your code with hacks just because Visual Studio gratuitously refuses to use the same function names as everyone else?

1. Can't use `main` as an entry point (because it has mangled arguments - need to ignore them and do it again properly from `GetCommandLineW`)
2. Can't use eg. `fopen`, need to ifdef in `_tfopen` just for Windows.
3. Tons of places you need to add underscores, call "_s" versions, and so on. It's pure lock-in, a complete pain.
4. Gratuitous function names - `_tcsnicmp` (`strncasecmp`) and `_ftelli64` (`ftello`). Can we really not provide the same names as the rest of the world?

### A note on UTF-8

Did I mention UTF-8? It's the modern text encoding that's the *right way* to handle characters. [Read the UTF-8 Manifesto](http://www.utf8everywhere.org/). The wide character `wchar_t` interfaces in ISO C are at best useless cruft now, and don't achieve their purpose because `wchar_t` doesn't hold a single codepoint, in the Visual Studio CRT at least. It's not "the Windows tradition", but a sane modern implementation of C for the Win32 platform would either set `wchar_t` to be the same as `uint32_t`, so that applications can be source-compatible with Unix uses of `wchar_t` for a codepoint (rather than one half); or, the `wchar_t` interfaces could be completely omitted.

Scope and Status
----------------

I have only a rudimentary implementation so far. It's very incomplete, and I have no plans to implement all of ISO C (it's huge, I'm starting with the bits I use). I'm using FreeBSD's libc as a basis. Selected XOPEN or non-POSIX functions have also been added where they have clear advantages (such as `xlocale.h` from Darwin/glibc/FreeBSD).

### Open questions:
1. Will I ever get this working?
1. Will I add support for loads of POSIX functions?
1. Is there any community that could form around this?
