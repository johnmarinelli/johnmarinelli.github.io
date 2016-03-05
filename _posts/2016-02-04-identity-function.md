---
title: Why Do We Need An Identity Function?
updated: 2016-02-04
---

## Why do functional languages have the identity function?
If you're anything like me, you have asked this question over and over again, and maybe even blindly copied and pasted code snippets from a book that included an indentiy function without really knowing why.  
I have been reading through [The Pragmatic Programmer's Programming Clojure](https://pragprog.com/book/shcloj2/programming-clojure) and, upon coming to the following example:

```clj
(sort-by :grade > [{:grade 83} {:grade 90} {:grade 77}])
```

I was messing around, and I had wanted to generate the array of `{:grade}` objects using a map function.

```clj
; a cool thing about clojure (and most functional languages) is that they grok the concept of infinite lists pretty well.
(def whole-numbers (iterate inc 1)) ; => (1 2 3 4 5 6 ... inf)
```

This was my original code:

```clj
(sort-by :grade > (map #({:grade %}) (take 10 whole-numbers))) ; => ArityException Wrong number of args (0) passed to: PersistentArrayMap
```

To me, the `(map #({:grade %}) (take 10 whole-numbers))` should have given:
`[{:grade 1} {:grade 2} {:grade 3} ...]`
However, the reader macro `#()` actually expands to `(fn [%1] ({:grade %1}))`, which wraps the function literal in parenthesis.

What I really wanted was `(fn [%1] {:grade %1})`, which really doesn't make any  sense once you think about it.
In fact, you "can't have any literal value as the body of an anonymous function", because it will get wrapped up in parenthesis.
The solution: Use identity!  

```
(sort-by :grade > (map #(identity {:grade %}) (take 10 whole-numbers)))
```

This unwraps to `(fn [%1] (identity {:grade %1}))`, and identity acts as a wrapper for our literal.  

Another reason is for when you have a function that returns the application of another function to some input:

```clj
(defn add-ten-and-do [fn in] (f (+ 10 in)))

(add-ten-and-do str 10) ; => "20"
```

What if you just wanted to add ten?

```clj
(add-ten-and-do identity 10) ; => 20
```

Finally, a cool trick with identity is to `filter` out `nil`s.

```clj
(filter identity [1 2 3 nil nil 6]) ; => (1 2 3 6)
```


[Reference SO question](http://stackoverflow.com/questions/2020570/common-programming-mistakes-for-clojure-developers-to-avoid)
[More uses for identity](http://www.dreamincode.net/forums/topic/234900-id-in-haskell/)
