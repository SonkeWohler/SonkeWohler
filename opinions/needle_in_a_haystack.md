# Thoughts on Finding a Needle in a Haystack

As every good programmer knows there are about three sensible approaches to
finding a needle in a haystack.

* You can sift through the stack and pray that the user doesn't refresh the
  page or kill the process before you are done.
  - An improvement on this would be a magnetic pitchfork, but that's still
    *O(N)* time where *N* is the number of straws in the stack.  And there is a
    lot of straw.
* You can be there when the needle is placed in the stack and remember roughly
  where it was.  I guess that is at best *O(log N)* time.
  - Even better you place it on its separate stack of needles to later
    retrieve in constant time.  Assuming needles are all identical anyway.
* And lastly, you buy a new needle and pretend its the one you were looking for
  in the stack.

Meanwhile the engineer will probably just throw the whole stack in water and
run a magnet along the bottom of the tub, depending on whether you need the
straw later.
