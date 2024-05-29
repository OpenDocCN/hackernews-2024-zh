<!--yml
category: 未分类
date: 2024-05-29 13:26:01
-->

# on the impossibility of composing finalizers and ffi — wingolog

> 来源：[https://wingolog.org/archives/2024/02/26/on-the-impossibility-of-composing-finalizers-and-ffi](https://wingolog.org/archives/2024/02/26/on-the-impossibility-of-composing-finalizers-and-ffi)

While poking the other day at making a Guile binding for [Harfbuzz](https://harfbuzz.org/), I remembered why I don’t much do this any more: it is impossible to compose GC with explicit ownership.

Allow me to illustrate with an example. Harfbuzz has a concept of *blobs*, which are refcounted sequences of bytes. It uses these in a number of places, for example when loading OpenType fonts. You can get a peek at the blob’s contents back with [`hb_blob_get_data`](https://harfbuzz.github.io/harfbuzz-hb-blob.html#hb-blob-get-data), which gives you a pointer and a length.

Say you are in LuaJIT. (To think that for a couple years, I wrote LuaJIT all day long; now I can hardly remember.) You get a blob from somewhere and want to get its data. You define a wrapper for `hb_blob_get_data`:

```
local hb = ffi.load("harfbuzz")
ffi.cdef [[
typedef struct hb_blob_t hb_blob_t;

const char *
hb_blob_get_data (hb_blob_t *blob, unsigned int *length);
]]

```

Presumably you then arrange to release LuaJIT’s reference on the blob when GC collects a Lua wrapper for a blob:

```
ffi.cdef [[
void hb_blob_destroy (hb_blob_t *blob);
]]

function adopt_blob(ptr)
  return ffi.gc(ptr, hb.hb_blob_destroy)
end

```

OK, so let’s say we get a blob from somewhere, and want to copy out its contents as a byte string.

```
function blob_contents(blob)
   local len_out = ffi.new('unsigned int')
   local contents = hb.hb_blob_get_data(blob, len_out)
   local len = len_out[0];
   return ffi.string(contents, len)
end

```

The thing is, this code is as correct as you can get it, but it’s not correct enough. In between the call to `hb_blob_get_data` and, well, anything else, GC could run, and if `blob` is not used in the future of the program execution (the continuation), then it could be collected, causing the `hb_blob_destroy` finalizer to release the last reference on the blob, freeing `contents`: we would then be accessing invalid memory.

Among GC implementors, it is a truth universally acknowledged that a program containing finalizers must be in want of a segfault. The semantics of LuaJIT do not prescribe when GC can happen and what values will be live, so the GC and the compiler are not constrained to extend the liveness of `blob` to, say, the entirety of its lexical scope. It is perfectly valid to collect `blob` after its last use, and so at some point a GC will evolve to do just that.

I chose LuaJIT not to pick on it, but rather because its FFI is very straightforward. All other languages with GC that I am aware of have this same issue. There are but two work-arounds, and neither are satisfactory: either develop a deep and correct knowledge of what the compiler and run-time will do for a given piece of code, and then pray that knowledge does not go out of date, or attempt to manually extend the lifetime of a finalizable object, and then pray the compiler and GC don’t learn new tricks to invalidate your trick.

This latter strategy takes the form of [“remember-this” procedures that are designed to outsmart the compiler](https://github.com/ivmai/bdwgc/blob/master/include/gc/gc.h#L1469-L1491). They have mostly worked for the last few decades, but I wouldn’t bet on them in the future.

Another way to look at the problem is that once you have a system working—though, how would you know it’s correct?—then you either never update the compiler and run-time, or you become fast friends with whoever maintains your GC, and probably your compiler too.

For more on this topic, as always Hans Boehm has the first and last word; see for example the 2002 [*Destructors, finalizers, and synchronization*](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=ac6669e60ef9ae9adbf3e596377d731584c27049). These considerations don’t really apply to *destructors*, which are used in languages with ownership and generally run synchronously.

Happy hacking, and be safe out there!