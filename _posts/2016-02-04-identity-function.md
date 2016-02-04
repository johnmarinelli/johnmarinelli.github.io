---
title: Why Do We Need An Identity Function?
updated: 2016-02-04
---

## Why do functional languages have the identity function?
If you're anything like me, you have asked this question over and over again, and maybe even blindly copied and pasted code snippets from a book that included an indentiy function without really knowing why.  
I have been reading througha [The Pragmatic Programmer's Programming Clojure](https://pragprog.com/book/shcloj2/programming-clojure) and, upon coming to the following example:

```clj
(sort-by :grade > [{:grade 83} {:grade 90} {:grade 77}])
```

I had wanted to generate the `[{:grade 83} {:grade 90} {:grade 77}]` using a `map` function along with an array of numbers.
[this stackoverflow answer helped a ton](http://stackoverflow.com/questions/2020570/common-programming-mistakes-for-clojure-developers-to-avoid)

```clj
; a cool thing about clojure (and most functional languages) is that they grok the concept of infinite lists pretty well.
(defn whole-numbers (iterate inc 1)) ; => (1 2 3 4 5 6 ... inf)

; This was my original code
(sort-by :grade > (map #({:grade %}) (take 10 whole-numbers))) ; => ArityException Wrong number of args (0) passed to: PersistentArrayMap
; To me, the (map #({:grade %}) (take 10 whole-numbers)) should have given:
; [{:grade 1} {:grade 2} {:grade 3} ...]
; However, the reader macro #() actually expands to (fn [%1] ({:grade %1})), which wraps the map literal in parenthesis.
; In fact, you "can't have any literal value as the body of an anonymous function", because it will get wrapped up in parenthesis.
; The solution: Use identity!  
(sort-by :grade > (map #(identity {:grade %}) (take 10 whole-numbers))) ; => [ {:grade 10} {:grade 9} {:grade 8} ...]
; The mathematical definition of identity is given a function f(x), return x.
; so, we essentially "unwrap" the 'x' from its parenthesis.

```

