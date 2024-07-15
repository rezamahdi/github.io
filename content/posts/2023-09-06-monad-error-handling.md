---
title: Monad Error Handling in C++
taxonomies:
  tags: [c++]
---

Well, finally I faced to a new problem that I love :-).

In many sources, like [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) and
[LLVM Coding Style](https://llvm.org/docs/CodingStandards.html), there is some nice points that there is no
good in using C++ Exceptions becaus it can be computionaly heavy, space consuming and also may cause design
mis-behavior.

So what is the alternative? well, it is your choise! You can use C-ish error handling, or follow LLVM and Google.

As the google projects, like chromium and gRPC, and LLVM itself
are really grate works, and are VERY ACTIVELLY maintain, there must be advices in them that how error handling is done.

## C-ish modernized

In projects like [RocksDB](http://github.com/facebook/rocksdb) and [LevelDB](http://github.com/google/leveldb),
that are following google style, functions return a unfied class named `Status`. In fact, it is built from other
classes that in sum, form the `Status`. by the way, it indicates any error (and also *success*) that is occured
during function call. it is just a try, to modernize the C-ish *error code* concept, nothing else...

It can convert error code to `std::string` and evalutaed in an `if` block; but there is no sence that it have
anything more than an *error code*.

Nice try! and I love it (In any good C++ developer, there is a C programmer that wnats to rip the skin out!). but
there is something about this mechanism that is annoying: *Result of execution*. if you are familiare with leveldb
(or rocksdb), this code is meaningfull:

```cpp

  leveldb::Options options;
  leveldb::Status status;
  leveldb::DB *db;

  status = leveldb::DB::Open(options, "path", &db);
  
  if(!status.ok()) ...
```

The result of `leveldb::DB::Open` is the opened database, but it is on argument list of function! I personally prefer
to get the reult as return code, becaus it can be chaind, and passed to other function in-place. like this:

```cpp
  InitObjectStore(leveldb::DB::Open(option, "path"));
```

... and handle error there also. I know... this is not a good reason.

## Monad (LLVM) Style Error Handling

We just pointed that return value of a function may be an error, or the result (are we?). So what?
error is very different with value, these are two different types! are you kiding?

You are right. a function can have just one return type. so we implement a class, to save both!
in LLVM it is `Expected<T>` class. it can hold a reference to a `T` or an `Error`;
just go to [llvm/Support/Error.h](https://github.com/llvm/llvm-project/blob/main/llvm/include/llvm/Support/Error.h).
You can modify it for you project (watch the license), and use it freely.

Well, let me show some magic!

The central part of this *library* is the `Error` class. It is just a pointer to a `ErrorInfo`,
that holds error information, and a boolean to track error handling. If you don't handle an error, when stack unwinds,
the destructor of `Error` will print some messages and terminate your program! This way you will inform that there
is an unhandled error in a scope.

Note that in contrast to exceptions, an `Error` doesn't propagate to upper scope, so any error must be handled in
current scope, or returned to caller of function. Let's sum up some properties of errors:

1. any error that is produced MUST be handled regradless of ignorability. this may seem so verbose and
   unpleasured, but it *force* the developer to write exception-safe codes (as possible).

2. Any error must be handled it scope that is produced or return to uper scope. Pay attenstion to
   *scope*. objects will be destructed at the end of scope, so an error must be handled in ``if`` block either:
   
   ```cpp
     ...
     if(true) {
       Error err = DoSomething();
      } // err is not checked here!
     ...
   ```
   
3. `Error`s can be evaluted as booleans, `true` means error, `false` means OK!

4. Custom errors can be defined by subclassing from `ErrorInfo<T>`. Note that it is a CRTP class:

   ```cpp
  
     class ResourceAllocationError
       : public ErrorInfo<ResourceAllocationError> {
      public:
       static char ID;
       std::error_code convertToErrorCode() const override {
         return std::make_error_code(
                     std::errc::resource_unavailable_try_again
                );
       }

       std::string message() const override {
         return "Can't Initialize Resource";
       }
     };
   ```

Error or Value?
---------------

If we expect a function return value, but it may fail, we use `Expect<T>` as return type of function.

```cpp
  Expected<int> div(int a, int b) {
    if(b == 0)
      return make_error<DivideByZeroError>("b is 0");
    return a/b;
  }
```

If an `Expected<T>` contains any erro, it is accessable by `takeError` method. Note that `Expect<T>` are
convertable to boolean as well, but opposite of `Errors`.

Correct way of handling errors in this style is using this method:

1. `handleErrors`: that takes error as first argument, and some handlers
   for any possible error. Handler is a function that may or may not return an error, and takes just one argument that
   its type is error type that can handle. if there is no handler for error, or any new error occure during handling,
   it will be returned. On successfull handling, a ``SuccessError`` will be returned.

2. `handleAllErrors`: is just same as above, except that it doesn't return
   anything.

3. `handleExpected`: it is useful for handling `Expected<T>`. if expected
   contains any error, handlers will be called like above, otherwise the value will be returned.

## Conclusion

This method of error handling is more robust than C-ish style or C++ Exception, but there is something about it that
make it not applicable in all purposes: it requires a *clean*, *mean*, *lean* and *wise* design in your project!
It doesnt seems so limitation, exceptions requires this also, by the way...

Other thing about this method is in handling constructor method errors. constructors doesn't return any value. so you
can't return `Error` objects. If a constructor may fail (it occures rarely), use factory method.

In next posts, I will show strategies that may help handling errors using Monad error handling.

