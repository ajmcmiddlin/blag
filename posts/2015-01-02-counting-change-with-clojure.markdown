---
author: Andrew
comments: true
date: 2015-01-02 10:00:03+00:00
layout: post
link: http://ajmccluskey.com/2015/01/counting-change-with-clojure/
slug: counting-change-with-clojure
title: 'Counting change with Clojure: A dalliance in dynamic programming'
wordpress_id: 558
categories: Programming
tags: clojure dynamic programming memoization SICP
---

I'm in the process of doing a great [bioinformatics course](https://www.coursera.org/course/bioinformatics) on Coursera. One of the topics covers some dynamic programming, and uses making change as a toy problem to demonstrate the problem that dynamic programming solves, and how to implement it. I had fun remembering this stuff, and thought I'd do a little write up on how to solve a toy problem like this using Clojure. In the interest of not directly providing solutions to coursework, we'll solve the related problem in [SICP of counting change](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-11.html#%_sec_1.2.2).

<!-- more -->



## The problem


Rather than go into too much detail on the problem, I'll leave you to read the section in SICP linked to above. Furthermore, I recommend that you take a look at [Bill the Lizard's write up on the orders of growth of this problem](http://www.billthelizard.com/2009/12/sicp-exercise-114-counting-change.html) to see what the problem with performance is. Essentially it boils down to the fact that the simple/naïve recursive solution requires solving exactly the same sub-problem a surprising number times - even in a relatively small example.

Before we go any further, here's the naïve solution implemented in clojure.


    
    <code class="brush: clojure">
    (defn count-change
      [amount coins]
      (cond (= amount 0) 1
            (or (< amount 0) (zero? (count coins))) 0
            :else (+ (count-change amount (rest coins))
                     (count-change (- amount (first coins)) coins))))
    
    (time (count-change 500 [1 5 10 25 50]))  ; =>  1965.408ms
    (time (count-change 750 [1 5 10 25 50]))  ; => 10395.13ms
    (time (count-change 1000 [1 5 10 25 50])) ; => 38044.189ms
    </code>



 A diligent reader might note that SICP suggests `(count-change 100 [1 5 10 25 50])` will take a while, but we're making allowances for the effect of [Moore's law](https://en.wikipedia.org/wiki/Moore%27s_law) over the last 30 odd years since SICP was first published. In any event, this solution obviously does not scale. The question is, how do we fix it?

 

## Solution 1: Clojure's memoization


 Memoization is an obvious solution to this problem, and is one put forward in a [footnote](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-11.html#footnote_Temp_54) in SICP. Lucky for us, Clojure will do this for us with its handy dandy memoization function, [`memoize`](http://conj.io/store/org.clojure/clojure/1.6.0/clojure.core/memoize/). We just pass `memoize` our function, and get back a memoized version of it. What could be easier? Not only that, but [memoize's source code](https://github.com/clojure/clojure/blob/eccff113e7d68411d60f7204711ab71027dc5356/src/clj/clojure/core.clj#L6018) is dead simple if you want to see exactly how it's implemented.


    
    <code class="brush: clojure">
    (def memoized-cc (memoize count-change))
    (time (memoized-cc 750 [1 5 10 25 50])) ; => 10258.13ms
    </code>



Uh oh. We memoized our function, but it only gave us a 1.3% boost in speed. Turns out there's a gotcha when memoizing a recursive function: you must rebind the recursive function to its memoized version. This makes total sense. We called `memoized-cc`, but every recursive call was to `count-change`, which is not memoized. To prove it, look what happens when we call `memoized-cc` again with the same arguments.


    
    <code class="brush: clojure">
    (time (memoized-cc 750 [1 5 10 25 50])) ; => 0.039ms
    </code>



Much better. So, let's rebind `count-change` to our memoized version and see how things go. Note that the namespace was reloaded before each call to `count-change` below so that memoized results didn't speed up subsequent evaluations.


    
    <code class="brush: clojure">
    (def count-change (memoize count-change))
    (time (count-change 500 [1 5 10 25 50]))  ; =>  5.175ms
    (time (count-change 750 [1 5 10 25 50]))  ; =>  7.175ms
    (time (count-change 1000 [1 5 10 25 50])) ; => 10.61ms
    </code>



Awesome! Let's see how far we can push our amount value until the wait gets unbearable.


    
    <code class="brush: clojure">
    (count-change 10000 [1 5 10 25 50])
    ; => CompilerException java.lang.StackOverflowError
    </code>



Oh dear. It turns out that using our recursive algorithm with memoization is fast, but as amount grows so does the stack of recursive calls we need to make. If amount gets too big, the stack is blown. If only our recursive calls didn't go so deep.



## Solution 1b: Clojure's `memoize` and count upwards


As we've just seen, once amount gets too large we'll blow our call stack. However, we can take further advantage of the fact that our function is now memoized. It turns out, if we simply start at 0 and calculate all of the ways we can make change for ever increasing amounts, by the time we get to a stack-blowing amount, our memoization cache is all warmed up and what were once deep recursive calls become cache hits.


    
    <code class="brush: clojure">
    (defn count-change-up
      [amount coins]
      (->> (range (inc amount))
           (map #(count-change % coins))
           last))
    
    (time (count-change-up 1000000 [5 10 20 50 100 200])) ; => 24182.815
    </code>



Success! We can now work out how many ways we can make change for large amounts of money. This is obviously very useful and important information, and it's a great relief to finally have it. On the downside, we are possibly performing more computations than we need to given that we don't necessarily need to know the value for every amount and coin combination between 0 and our final amount. However, the complication and overhead of working out which computations are strictly necessary, and then performing them in increasing order of amounts, is a little beyond me right now. My gut tells me any solution I did come up with would be a net loss in performance anyway. At least we're getting timely answers!



## Solution 2: Algorithmic memoization


I'm not sure that _algorithmic memoization_ is the right term here, but I'm going with it. What I mean by it is modifying the algorithm itself to incorporate its own, specific form of memoization. Why would we bother? While it would be trivial to implement Clojure's version of `memoize` if one weren't provided, it's a generic solution that isn't necessarily optimal for our problem or requirements. "But Andrew, [premature optimization is the root of all evil](https://en.wikipedia.org/wiki/Program_optimization#When_to_optimize)", I hear you retort. I'm not disputing that, but this is an academic exercise, so without a real set of requirements to satisfy we're just going to explore our options. Just for fun. This is fun right?

To add a touch of concreteness to this very fuzzy beginning, in return for this vague hope that rolling our own memoization will have benefits, the downsides include:




  
  1. The code becomes more complicated as most of it is now managing a cache rather than achieving its goal.

  
  2. Multiple threads calling a memoized function all contribute to, and benefit from, the one cache - this is no longer possible.

  
  3. The cache is no longer persistent across calls, so repeated calls to the function with the same arguments will always need to recalculate their results.



Enough blabbing. Code.


    
    <code class="brush: clojure">
    (defn- cc-add-to-cache
      [amount coins cache]
      (->> (map (partial -' amount) coins)
           (map-indexed #(get-in cache [%2 %1] 0))
           (reductions +')
           (map-indexed list)
           (reduce (fn [[c [coins-used value]]]
                     (assoc-in c [amount coins-used] value))
                   cache)))
    
    (defn count-change-algomemo
      [amount coins]
      (let [max-coin (apply max coins)
            coin-count (count coins)]
        (loop [a 1
               cache {0 (zipmap (range (count coins))
                                (repeat 1))}]
          (if (> a amount)
            (get-in cache [amount (dec coin-count)])
            (->> (cc-add-to-cache a coins cache)
                 (recur (inc a)))))))
    
    (time (count-change-algomemo 1000000 [5 10 20 50 100 200])) ; => 11841.47ms
    </code>



We've definitely lived up to the more complicated code part, but I don't think it's unwieldly. It can be improved too, as we'll see (suggestions also very welcome). It also appears faster, **but please note**; while I tried to draw a more meaningful performance comparison between the two, I got some erratic results, so I'm not convinced that this version is faster than `count-change-up`. Sometimes `count-change-up` seemed faster, sometimes it didn't.

All that effort, and we have crappier code and the same performance. Great job team. I guess now would be a good time to find a benefit to this approach. Conveniently, I have one ready to go: memory usage. `count-change-up` and `count-change-algomemo` should both be similar in terms of their memory usage - at least while executing. They both keep a cache of _all_ values for amounts between 0 and amount, so the space requirement is proportional to `(* amount (count coins))` for both. Depending on your use case, this may be a lot of wasted memory. If you're running this function often, then maybe it is worth using Clojure's `memoize` and holding onto already calculated values. However, if you're short on space or don't need to hold onto previous results, using our custom cache allows us to calculate results in constant space proportional to the size of the greatest coin denomination. This is an option you simply don't get with Clojure's built in `memoize`. It should be noted at this point that I'm flagrantly ignoring garbage collection and how that works with memoization caches because I don't want to go there.

To see how we can keep space constant with regard to the maximum coin value, we make the observation that the furthest back into the cache we ever look is `(- amount (apply max coins))`. This means that, for the purposes of calculating our result, we only need to hold onto the last `(apply max coins)` values in our cache. Luckily the problem of cache invalidation in this situation is a simple one, and we can easily fit it into our existing code.


    
    <code class="brush: clojure; highlight: [1, 2, 3, 19]">
    (defn- cc-cull-cache
      [amount max-coin cache]
      (select-keys cache (range (- amount max-coin) (inc amount))))
    
    (defn- cc-add-to-cache 
      ; same as before
      )
    
    (defn count-change-algomemo
      [amount coins]
      (let [max-coin (apply max coins)
            coin-count (count coins)]
        (loop [a 1
               cache {0 (zipmap (range (count coins))
                                (repeat 1))}]
          (if (> a amount)
            (get-in cache [amount (dec coin-count)])
            (->> (cc-add-to-cache a coins cache)
                 (cc-cull-cache a max-coin)
                 (recur (inc a)))))))
    
    (time (count-change-algomemo 1000000 [5 10 20 50 100 200])) ; => 82961.341ms
    </code>



This version is obviously less performant, but as is usually the case we're trading time for space. However, given that we now control the cache, we can try speeding it up with transient maps to get better time performance while keeping our lower memory usage. Given that some of the convenient map functions like `assoc-in` and `select-keys` aren't available for transient maps, we've had to switch to `assoc!` and `dissoc!`. This actually got me thinking about the code a bit more and resulted in cleaner code.


    
    <code class="brush: clojure; highlight: [3, 10, 11, 18]">
    (defn- cc-cull-cache
      [amount max-coin cache]
      (dissoc! cache (dec (- amount max-coin))))
    
    (defn- cc-add-to-cache
      [amount coins cache]
      (->> (map (partial -' amount) coins)
           (map-indexed #(get-in cache [%2 %1] 0))
           (reductions +')
           (zipmap (range))
           (assoc! cache amount)))
    
    (defn count-change-algomemo
      [amount coins]
      (let [max-coin (apply max coins)
            coin-count (count coins)]
        (loop [a 1
               cache (transient {0 (zipmap (range (count coins))
                                           (repeat 1))})]
          (if (> a amount)
            (get-in cache [amount (dec coin-count)])
            (->> (cc-add-to-cache a coins cache)
                 (cc-cull-cache a max-coin)
                 (recur (inc a)))))))
    
    (time (count-change-algomemo 1000000 [5 10 20 50 100 200])) ; => 5395.502ms
    </code>



Look at that! Constant, and comparatively small memory usage with better time performance. Maybe there is a use case for rolling your own memoization. Just to be clear though, these performance figures are potentially wildly inaccurate, and in the case of the memory usage, purely theoretical. Having said that, I still had fun and learned a lot, and hopefully you got something out of it too. If not, have this: there are 41,707,305,668,679,272,501 (over 41 quintillion) different ways to make change for a million dollars using only Australian coins. Ponder this next time you think your cashier is a little slow deciding how to give you change.

