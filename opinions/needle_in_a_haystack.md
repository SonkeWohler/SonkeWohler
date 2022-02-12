# Thoughts on Finding a Needle in a Haystack

As every good programmer knows there are about three sensible approaches to
finding a needle in a haystack.

### Search

You can sift through the stack and pray that the user doesn't refresh the page
or kill the process before you are done.

- An improvement on this would be a magnetic pitchfork, but that's still
  *O(N)* time where *N* is the number of straws in the stack.  And there is a
  lot of straw.
- You get even more improvement if you can parallelize this, but it's still 
  *O(N)*.
- To be clear *O(N)* wouldn't be **that** bad if there wasn't so much straw.
- Yes, I'm assuming here that we can't represent the stack with a data
  structure that would make this better.

### Preemptive Sort

You can be there when the needle is placed in the stack and remember roughly
where it was.  I guess that is at best *O(log N)* time.

- Even better you place it on its separate stack of needles to later
  retrieve in constant time.  Assuming needles are all identical anyway.
- If you do this after you used the first solution it's called *caching*.
- If you do this before you ever did the work of searching through the stack
  it is called *read optimization*.

### Lying to the User

And lastly, instead of doing the work of either of the above, you can always
just go out and buy a new needle and pretend its the one you were looking
for in the stack.

## Meanwhile

While you do that the engineer will probably just throw the whole stack in
water and run a magnet along the bottom of the tub, depending on whether you
need the straw later.
