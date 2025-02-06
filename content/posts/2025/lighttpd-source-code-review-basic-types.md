+++
title = "Lighttpd Source Code Review - Basic types"
date = 2025-02-06

[taxonomies]
tags = ["c", "http", "server"]
+++

Lighttpd uses modular design. In programs like web servers, that uses many data structures for both
optimization and ease of develop, common choice is to define this data structures like utilities.
Apache uses Apache portable runtime (APR), Nginx defines them in `src/core` subdirectory and lighttpd
also defines them. As the source code of lighttpd is not complicated as other web servers, source
files of this type are reside in `src` beside other sources.

<!-- more -->

Basic types in lighttpd are composed as:

 1. Utility types
   - **array**. Defined in `array.h`, as a simple array to contain same type objects.
   - **buffer**, Defined in `buffer.h`, a controlled container of mainly text data.
 2. Core types
   - **request**
   - **response**
   - **server**
   - **server_config**
   - **connection**

## Buffer
Buffer is a key type in lighttpd, it contains string, data and represents some meaning about them.
it's definition as as folow:

```c
/* generic string + binary data container; contains a terminating 0 in both
 * cases
 *
 * used == 0 indicates a special "empty" state (unset config values);
 * ptr might be NULL, too.
 *
 * copy/append functions will ensure used >= 1
 * (i.e. never leave it in the special empty state)
 */
typedef struct {
   char *ptr;
   /* "used" includes a terminating 0 */
   uint32_t used;
   /* size of allocated buffer at *ptr */
   uint32_t size;
} buffer;

```
As the comment, buffer is a generic data container for both string and data, and _a terminating NULL_.
field `ptr` points to the real data in memory and if the buffer is empty, it _may_ be NULL. Empty
state has different meaning in different situations. The comment says it is used to indicate _unset_,
may be for configuration file's options. By the way, the standard way to check empty state is to use
`used` field that saves the realy used amount of memory. `size` field indicates the total memory
allocated for this buffer, so the used memory may be lower than allocation.

Buffer operation includes many functions like init, free, move, copy, append, extend and many other.
String operations are implemented into buffer itself, but data operations are mainly implemented in
array type as array uses buffer as it's back. Read the `buffer.h` and it's comment to understand
supported operations.

## Array
Array uses buffer as it's underlying memory. This let's array to use buffers abstraction, buffer is
just a countineous chunk of memory, anyway.

implementation of array is based on 3 auxilary types: `struct data_unset`, `struct data_method` and
`enum data_type_t`. In fact, array of every special type, has it's own structure, controlled by
`DATA_UNSET` macro's fields:

```c
struct data_unset; /* declaration */
 
struct data_methods {
    struct data_unset *(*copy)(const struct data_unset *src); \
    void (*free)(struct data_unset *p); \
    void (*insert_dup)(struct data_unset *dst, struct data_unset *src);
};
 
typedef enum { TYPE_STRING, TYPE_ARRAY, TYPE_INTEGER, TYPE_CONFIG, TYPE_OTHER } data_type_t;
#define DATA_UNSET \
    buffer key; \
    const struct data_methods *fn; /* function table */ \
    data_type_t type
```

Every array struct, has `DATA_UNSET` as it's first line, that adds a buffer named _key_ as backend,
a method table named _fn_ and a field to distinguish type of objects stored in array.
