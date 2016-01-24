---
title: Laziness In Clojure
updated: 2016-01-24
---

## Laziness In Clojure

Lately, I've been playiing with [Clojure](http://clojure.com).  I've always wanted to work with functional languages, and Clojure has an awesome live-coding music library called [Overtone](https://overtone.github.io) that I'd like to learn.

I picked up [The Pragmatic Programmer's Programming Clojure](https://pragprog.com/book/shcloj2/programming-clojure) and learned a few basic concepts - defining forms, basic syntax, etc.  I am slowly starting to realize why everyone hails Lisp as ['the language from which God wrought the universe'](https://xkcd.com/224/).  

## The Problem
For my first Clojure project, I wanted to write a script that uses [the wikimedia API](https://www.mediawiki.org/wiki/API:Main_page) and stores all Wikipedia page titles and their ID in a database.  I'm also doing a little project on speeding up ActiveRecord queries, and a friend of mine approached me with a project that used millions of Wikipedia entries, so I figured I'd make my own database to tinker with.

However, when running this part of the script:

```clj
(defn insert-page-into-db [page]
  (try
    (sql/insert! db-conn-vars :pages
      [:title, :pageid]
      [(get page "title"), (Integer. (get page "pageid"))]) (catch Exception e (println e))))

(defn insert-pages-into-db [pages]
  (map #(insert-page-into-db %1) pages))

(defn get-wikimedia-pages [apfrom]
  (let [body (get-body (get-json-response (build-url apfrom)))]
    (let [continue-from (get-continue-from body) pages (get-pages body)]
      (insert-pages-into-db pages) ; <==== I never do anything with the result of this
      (if (not (nil? continue-from))
        (recur continue-from)))))
```

`#insert-page-into-db` would not get run.  I littered `println` statements everywhere, but all that I could gather was that `#insert-page-into-db` was simply not getting ran.  I had a few suspicions, but being a Clojure newbie I sought help in freenode#clojure.  One of my suspicions was that since I never actually evaluate the return array of the `map`, it never gets called.  After having my suspicions validated by a helpful IRCer, I changed my `map` to `run!`, which is like `map` but it expects side effects and doesn't return a modified collection.  You can think of it as Ruby's `Enumerable::each`.

```
(defn insert-pages-into-db [pages]
  ; run! is an unlazied map because it is ran for the purposes of side effects
  (run! #(insert-page-into-db %1) pages))
```

I understood the core concept; if an object isn't going to be used later, don't bother wasting cycles creating said object.  Clojure is considered 'partially lazy' - unlike, for instance, Haskell, Clojure only implements lazy evaulation for the majority of its sequence operations, where Haskell covers the whole spectrum.


> ### If an object isn't going to be used later, don't bother wasting cycles creating said object.

The largest pro of lazy evaluation is the performance implications.  Suppose we have a function that randomly generates 10 widgets, and we create 1,000 "objects" that make use of this function.  With eager evaluation, this turns out to be 10,000 widgets, which could be expensive.  With lazy evaluation, those 10 widgets *won't be evaluated until inspection*.  For example, until I say `(filter #(+ %1 1) (get object :widgets))`, the `object`'s widgets *do not exist*.  Pretty sweet.  

This also means that control structures are saved needless evaluation as well.  In an eager-evaluated language,  `if a then b else c` would evaluation both `b` and `c`.  However, with lazy evaulation, only one of `b` or `c` will be evaluated.  Conceptually, it's similar to a boolean short circuit `||=`.

Finally, on the more theoretical side of things, lazy evaluation allows for *infinite data structures*.  Sounds fancy!  
What this really means is that we can define data in terms of unending recursion, and simply take N pieces of that data as we please.  Of course, there is no realistic "infinite list"; however, as humans, we like to be able to express some things (like the number line) in infinite terms.

## Conclusion
I've been in the Clojure trenches for almost a week now, and have been really enjoying the functional style.  I've made a few attempts with Haskell before, but never really had a use case for any sort of functional wizardry.  Now, I'm really excited to start using [Overtone](https://overtone.github.io) and hopefully will get proficient enough to do live performance with it.  
