+++
title = "Lighttpd Source Code Review - Intro"
date = 2025-02-06

[taxonomies]
tags = ["c", "http", "server"]
+++

[Lighttpd](https://lighttpd.net) is a fast, efficient and popular web server. by it's efficiency, it is
simple too. In contrast to other high efficient and event driven servers, it is simply single-threaded.
Code base of lighttpd is built up by 123kloc, with all of it's modules included. In this post we want to
explore the source code and understand the architecture.

<!-- more -->

At first, lets fetch the source code from it's [repo](https://git.lighttpd.net/lighttpd/lighttpd1.4.git)

```sh
git clone https://git.lighttpd.net/lighttpd/lighttpd1.4.git lighttpd

```

Now, lets explore the directory hierarchy. Almost all of source files ar located at `src` directory. In
this directory there is 3 kinds of files:

 1. Basic utilties (`ck.h`, `first.h`)
 1. Core sources
 1. Modules' source files (any file with `mod_` prefix)

Before we start explore, lets talk about some coding convention in lighttpd. In all condition evaluations
the convention is that constant comes first. for example:
```c
if(NULL != srv.errh) {...}

/* instead of */

if(srv.errh != NULL) {...}
```

This convention is called _Yoda condition_. There is no difference between two convention, it just makes
sense in comparing equality. With yoda condition, compiler will make an error if programmer accidentally
_assign_ (using `=`) instead of _comparision_ (using `==`).

Next convention is to use function attributes extensively. Attributes are defined in `first.h` as macros
in order to make the source code more readable. Most used attributes are:

 1. `__attribute_noinline__` for marking function to not be inlined at all.
 1. `__attribute_returns_nonnull__` function doesn't return NULL value. So compiler may ignore 
  NULL-checking and optimize the code.
 1. `__attribute_cold__` marks the function as cold, means it is called rarelly.
 1. `__attribute_hot__` marks function as hot, means it is called frequently.

Setting function's hottness/coldness make compiler to place them toghether in a separate code section.
This makes an opportunity in OS's swap to keep the hot section in memory and swap the cold section as
it is not needed.

Definitions and macros defined in `first.h` is so important. Allmost all source files include it (as the
name says).

## Checked functions

Now, let's look at `ck.h` (aka. _check_). This file defines basic functions that may return errors. Some
functions that are defined here is as:

 1. `ck_malloc`, `ck_calloc`, ... for memory allocation as raw stdlib's but checks it's return value.
 1. memory comparison and manipulations, like `ck_memcp*`, `ck_memzero,...`
 1. Backtracing functions, like `ck_bt` for printing backtrace and `ck_bt_abort` to print and abort.
 1. Assertion utilities, `ck_assert`, `ck_assert_failed`, `ck_static_assert`.

As this functions are checked, they are marked with corresponding attributes for compiler optimization,
for example `ck_malloc` is marked as `__attribute_returns_nonnull__`. In all situations if assertion
failes, `ck_assert` is called (it is a macro) that in turn calls `ck_assert_failed` and it prints
backtrace to stderr and aborts.
