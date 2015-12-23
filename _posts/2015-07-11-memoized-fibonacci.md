---
title: Built-in memoization with Ruby hashes
updated: 2015-07-11
---

```ruby
fib = Hash.new{ |h, k| h[k] = k < 2 ? k : h[k-1] + h[k-2] }
```
