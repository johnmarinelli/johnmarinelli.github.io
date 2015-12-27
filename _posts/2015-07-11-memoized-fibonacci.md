---
title: Built-in memoization with Ruby hashes
updated: 2015-07-11
---
Ah, Ruby - you never cease to surprise and delight me.  In this short writeup I'll show you how to achieve memoization using a particular Hash constructor.

## Memoization
Memoization is a term used for caching results of some function.  Conceptually, you create a lookup table that 'remembers' the result of some computation given the same input.

```
y = some_expensive_function x # takes 5000ms
cache[x] = y

# ...

z = some_expensive_function x # O(1) time since we have the value of some_expensive_function 
                              # with input 'x' memoized
```

## Memoized Fibonacci

Take a look at this:

```ruby
fib = Hash.new{ |h, k| h[k] = k < 2 ? k : h[k-1] + h[k-2] }
```

Now, when you call fib(100), it will calculate the Fibonacci sequence up to 100.  However, it also stores all the Fibonacci values between up to 99 in the same hash!  Now any call to `fib n` where `0 <= n <= 100` will have O(1) time.

## Breakdown
The key ingredient in this is a certain Hash constructor that takes in a block as an argument.  The `{ |h, k| }` block is called with the hash object and the key, and allows you to modify the hash object dynamically.  The rest of the code is the familiar Fibonacci function.

## Conclusion
This little hack is useful for recursive mathematical functions; I've personally had some fun with this technique with [the logistic map function](https://en.wikipedia.org/wiki/Logistic_map), which is studied in the field of Chaos Theory due to its interesting chaotic properties.


