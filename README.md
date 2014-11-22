

Tiny Logger
-----------
THIS PROJECT IS UNDER DEVELOPMENT AND SHOULDN'T BE CONSIDERED STABLE NOW.

Another feature reduced asynchronous data logger. Sponsored by my employer **Diadrom AB.**

We just wanted an asynchronous logger that can be used from many dynamically loaded libraries without doing link-time hacks like linking static and hiding symbols and some other features.

After having maintained a slightly modified version of glog and given the fact that this is a very small project we decided that existing wheels weren't round enough.

## Design rationale ##

 - Simple
 - Asynchronous
 - Fast for the caller thread
 - Suitable for soft-realtime
 - File rotation-slicing
 - Should work with g++4.7 and VS 2010
 - Boost dependencies just for parts that will eventually go to the C++ standard.

## How does it work ##

When the user is to write a log message, the size is precomputed (in most cases the compiler will deduce it as a constant), then the memory is reclaimed either from the fixed size free list or the heap (configurable) and the message is serialized and passed to the worker thread queue.

The user thread doesn't format strings, just copies built-in type values and whole program duration C string pointers (Deep copies can be done if the user requires it) to the message and appends data for the worker thread to be able to decode it.

The messages are formatted by printf-style strings, where the formatting string is required to be a literal (not a const char*):

log_error ("the value of i is {} and the value of j is  {}", i, j);

The function is type-safe, but without "constexpr" there is no way to parse the formatting string at compile time so format errors are caught at run time.

## File rotation ##

The library can rotate fixed size log files.

Using the current C++11 standard files can just be created, modified and deleted. There is no way to list a directory, so the user is required to pass at start time the list of files generated by previous runs.


## Weaknesses ##

 1. No C++ ostream support.
 2. Limited formatting abilities (to keep the serialized message compact).
 3. No way to output runtime strings/ memory regions without deep-copying them.
 
The third point is the most restrictive for my liking, it's just inherent to the asynchronous/non-blocking design, there is no guarantee that the about the passed data lifetime.

It's possible to artificially increment the refcount of a shared_ptr by copying to a instance created using "placement_new" and to decrement it in the worker using the same trick, but the calling code would end up being ugly, so I keep this idea on hold.

> Written with [StackEdit](https://stackedit.io/).

