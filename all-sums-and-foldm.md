Digging through @paf31's Purescript by Example is no picnic. LINK
That is, there's no sandwich eating, barbecuing or frisbee throwing.
Instead the reader is exposed to typeclasses, ADTs and monads without an oxygen supply.

There's this thing called `foldM`; 
it may well be one of the reasons kids will learn Haskell at schools in the near future.
Just kidding, but it's still quite fascinating. Maybe.

So how do you, given an array of numbers, find all possible sums that adding any subset of those numbers yields?
Hey, that sounds like one of those weird coding contest problems!
Or something your Probability & Statistics professor was droning about.

Relax, we'll wade in, but no sharks.

Now how about you take a minute (an hour, a day, a weekend on the shore) here 
and try and solve this super standard problem in your language of choice (I guess you'll choose JavaScript anyway)?

Don't worry, no one will get ahead of you, take your time.
Actually you can simply open DevTools right here and start hacking right away.
See, I knew you'd choose JavaScript.

You're back? Good.
Let's see what you wrote.

First of all, you considered how many sums can an array of *N* numbers produce.
Let's call the numbers n<sub>i</sub>, so a sum encompassing all the numbers would look like:

n<sub>0</sub> + n<sub>1</sub> + n<sub>2</sub> + ... + n<sub>N</sub>

To get all other sums we just need to "strike out" numbers getting a different subset and, therefore, a different sum every time:

<s>n<sub>0</sub></s> + n<sub>1</sub> + n<sub>2</sub> + ... + n<sub>N</sub>

n<sub>0</sub> + <s>n<sub>1</sub></s> + n<sub>2</sub> + ... + n<sub>N</sub> 

<s>n<sub>0</sub></s> + n<sub>1</sub> + <s>n<sub>2</sub></s> + ... + n<sub>N</sub>

and so on.

Note that the sums you get are not necessarily unique.
What if you strike out all the numbers? 
You get a zero, which is reasonable to regard as a valid sum of an empty subset of numbers.

Next let's imagine our striking out is multiplying by zero and keeping a number in the sum is multiplying by one.

Hey, a binary linear combination!

No need to call the lifeguards, the water is too shallow to drown:

a<sub>0</sub>n<sub>0</sub> + a<sub>1</sub>n<sub>1</sub> + a<sub>2</sub>n<sub>2</sub> + ... + a<sub>N</sub>n<sub>N</sub>

Which sum is that? All of them!

What?

It's all of them, with a<sub>i</sub> taking value 0 or 1 and giving us a neat way to encode any number subset. 
The "all-encompassing" sum thus becomes, in terms of a-coefficients:

`1 1 1 ... 1`

And zero is of course all zeroes:

`0 0 0 ... 0`

Between those two edge cases lies every possible sum of numbers represented by a subset denoted by a binary number.

This arms us with a delicious way of thinking about the sums of N numbers as binary numbers of N digits.

This number is easily obtainable for any N:

2<sup>N</sup>

Let's try it out: `[1, 2, 10]`, N = 3 so we expect 2<sup>3</sup> = 8 different sums. Not a buffet but will do for a picnic:

| a's |sum|
|:---:|:-:|
|0 0 0|  0|
|0 0 1| 10|
|0 1 0|  2| 
|0 1 1| 12|
|1 0 0|  1|
|1 0 1| 11|
|1 1 0|  3|
|1 1 1| 13|

No matter how hard you try, you won't find a sum that's not listed in the table above.

BTW That's a Knapsack Problem set of numbers since all sums are one-of-a-kind. LINK

The formula for the number of sums also gives us an idea about the computational complexity.
It's clear that to obtain 8 unique numbers (as happened in our *worst* case), the program would have to perform at least 8 computational operations.

What your colleagues/peers hide from you is that exponential complexity is okay if it's not somewhere inside a multi-level menu React component.

You're bored and rather be on a blanket with your SO/kids/dog and a few sandwiches?

Let's write some code. 

Hmmm, how do we print a nice table like the one above?

We could just loop through numbers from 0 to 2<sup>N</sup>-1 and then get a<sub>i</sub> by doing a right bitwise shift...

a<sub>i</sub> = M >> i & 1, with M is in `[0..7]`

for 5:

5 is 101, so

a<sub>0</sub> = 5 >> 0 & 1 = 1 (10**1**)

a<sub>1</sub> = 5 >> 1 & 1 = 0 (1**0**1)

a<sub>2</sub> = 5 >> 2 & 1 = 1 (**1**01)

and it's all zeroes from here for the remaining a's.

IT'S A SHARK!! GOOD LORD IT'S A HUGE BLOOD-THIRSTY SHARK!! :shark:

Let's not get there, it certainly doesn't look pretty.

By using a loop to construct binary numbers we intend to use as masks, we lose those shiny single bits we need so much to be multipliers.
We get them tightly packed into numbers and have to use the box-cutter (`>>`) to get to individual bits.

What a chore.

Instead of decomposing binary numbers we'd rather be composing.
Think about binary numbers as sequences.
There are but two ways for any binary sequence to become a _longer_ binary sequence:
a `0` or `1` have to come over and be interested in hot-dogs enough to join the queue:

```
[C.M.O.T Dibbler]
     ?  
    / \  
   0   1
```

Now that was the first person craving sausage-in-a-bun. What about the guy after them? But whom? `0` or `1`? Let's consider both cases:

```
[C.M.O.T Dibbler]
      ?
    /   \
   0     1
  / \   / \
 0   1 0   1
```

Wait a minute, what is that??
That's a freaking binary tree! Helping us to obtain binary numbers! Who would have thought??

Indeed.

```
   x
  / \ 
 0   1
```

For any binary number represented by a non-leaf tree node _x_ there are exactly two options to form a _longer_ binary number/sequence: to add 0 or 1 to the end by following either the left or the right branch.
Leaves are where binary numbers end.

All branches/paths of a full (all non-leaves have both children) binary tree of height N are all possible binary numbers up to 2<sup>N</sup>-1.

Hang on to this concept, it will be revisited.

Meanwhile, let's write some code.

So if we got all possible binary sequences in that tree, we should easily make it into a tree of all possible sums, right?

That's right!

For [1, 2, 10]:
```
                0
              /   \
            /       \
          /           \
+1       0             1
       /   \          /   \
+2    0      2      1      3
     / \    / \    / \    / \
+10  0 10   2 12   1 11   3 13
```

In order to get the sums, we just need to walk all the paths until we reach the leaves.
See how we started with _sums_, moved on to _sequences_ and ended up with _paths_?
That kind of stuff often happens to you when you are doing functional programming and playing with computational abstractions.

You're still waiting for us to start coding?
Okay, okay, less talking, more typing, but don't expect it to be too long though.

A good thing to start with would be a function called `sums` that accepts an array of numbers.
What should it return? We could build our binary tree with simple 2-element arrays, holding left and right subtrees.
Let's use recursive calls for building those:

```js
function sums (numbers) {
     return [
          sums(), // left or "0" subtree
          sums()  // right or "1" subtree
     ]
}
```

What arguments go inside the calls? Obviously simply passing numbers won't cut it, we have to show some progress as we follow the computational paths. 

First, let's add another argument, `sum`, to our function to indicate the path tree node we're going through.
Its value for the very first `sums` call would be `0`, just as in the root of our tree.
For every function call we'll get next number from the array and calculate two possible next values of `sum`: with or without this number, that is, for `0` and `1` paths we take:


```js
function sums ([head, ...tail], sum = 0) {
     return [
          sums(tail, sum),        // left or "0" subtree
          sums(tail, sum + head)  // right or "1" subtree
     ]
}
```

See? The `numbers` array becomes "_remaining_ numbers sum up" and is destructured into `head` containing the first element (for current iteration) and `tail` containing the rest of them to be passed further down.

Now we only need to know when to stop this fun train.
And that is where we detect that we have reached our leaves or, in fact, actual sums.
It's quite easy, our "_remaining_ numbers" array should be empty, since its length is equal to the height of the tree!
This can be detected, for instance, by checking if `head` is undefined (also known as Saturday Morning Condition).

What do we do in this case? Let's simply return our "leaf" value: the sum.

```js
function sums ([head, ...tail], sum = 0) {
     if (head === undefined) {
          return sum;
     }
     return [
          sums(tail, sum),        // left or "0" subtree
          sums(tail, sum + head)  // right or "1" subtree
     ]
}
```

Ok, that should be enough characters, let's run it (you still have those DevTools open, haven't you?)
Elon Musk might have been less excited about Falcon9.

SCREENSHOT

Crockford's nostrils, it works! We got ourselves a scrawny binary sum bonsai, and it checks out!
Go check it with the table values, but it LGTM.

Notice how we ended up with leaf values only, other nodes being only nested containers for them.
The non-leaf nodes are like ancient fossils retaining the shape of recursive computation long after it ends.

So we advanced through the input array without preserving any previous tree "levels".

(A curious reader might find another solution that actually requires just one recursive call instead of two to go through the "levels" but let's leave it until your next picnic/subway commute).

Then we don't actually need the tree-like structure, since it is inherently present in the nature of our computation itself!

Let's bounce those arrays and produce just one, flat array with all the sums.
Right now if we consider the return type of `sums`, we might call it "Hmmm, nested arrays, whatever" or, on second thought, a _tree_. And sometimes _number_. Ugh.
We can achieve this if instead of producing new arrays we always flatten the ones we get from the "deeper" level.

`Array.prototype.concat` can help us with that:

```js
function sums ([head, ...tail], sum = 0) {
     if (head === undefined) {
          return sum;
     }
     return sums(tail, sum)                  // left or "0" subtree
            .concat(sums(tail, sum + head)); // right or "1" subtree

}
```

There's an error in this code. Can you spot it?

Exactly, when we reach return a final sum, we can't call `concat` on it, since it's just a number.
One of the ways it can be fixed is that we always return an array:

```js
...

if (head === undefined) {
     return [sum];
}

...
```

SCREENSHOT

The return type changed from _tree-or-number_ to definite _array_, which should warm your heart if you're into static typing.
But never mind that, the function works!
We can congratulate ourselves on a problem well solved.
Perhaps you found a better solution? Somewhere in this article there must be a hint or two on how to get in touch, I'd really appreciate that, thanks.

It's time to walk away whistling, check your twitter and then check your fridge for sandwich ingredients.
Maybe it's even not that late and the weather is nice?..

...hold on, what about that `foldM` thing?

You're already standing at your door, picnic basket in hand?

Let's Purescript.

Without any (further) dramatic device, let's see how beautiful and concise the solution of our sums problem is in Purescript:

```purescript
sums :: Array Int -> Array Int
sums = foldM (\x y -> [x, x + y]) 0
```

Bear in mind, that the first line only serves as the function signature (yes, Purescript compiler, just like social networks, expects you to tell it everything about your data).

Array of Ints goes in, Array of Ints goes out, simple enough.
Otherwise that's it, that's the entire code. And it works too.

The second line is something like this in JavaScript:

```js

const sums = foldM((x, y) => [x, x + y])(0);

```

All functions in Purescript are curried, so in order to pass them multiple arguments you have to make multiple columns.
This allows to omit any number of such calls, easily constructing partial functions.

So `fold` is sort of like `reduce` in JavaScript, but what's `foldM`?

Let's take a look at foldM's signature (the one for folding arrays):

```purescript
foldM :: forall m a b. Monad m => (a -> b -> m a) -> a -> Array b -> m a
```

Now, let's get back from behind your couch, take a deep breath and take another look.

You didn't think we could make it through without monads, did you?
I suggest you read either @paf31's chapter on monads, or LYAHFGL or this one: LINKS

Anyway, we've got three types at play here, `a`, `b` and `m` and `m` has to be a Monad.

`(a -> b -> m a)` is a typical reducer upgraded to return a monad: values of `a` (accumulator) and `b` (current value) go in, `a` contained in a monad `m` goes out.

In our case, it's this expression: `\x y -> [x, x + y]`
But `[x, x + y]` is an Array! Does it mean that `m a` is actually `Array a`??
Does it mean that Array is a _Monad_. 

You bet it is.

`a` is something to pass into the first call of the folding function (`0`, just like in the JS version).
So `a` is `Int`, therefore, `m a` is just a good old `Array Int`, an array of integers.

`Array b` represents the sequence that we want to fold -- our numbers array! We know that it's `Array Int` too.

`m a` stands for eventual folding result and obviously matches the return value of the folding function.
Since resulting array of sums is `Array Int` as well, it fits perfectly.

Substituting all the deduced types yields:

```purescript
foldM :: forall m a b. Monad m => (a -> b -> m a) -> a -> Array b -> m a
-- foldM :: (Int -> Int -> Array Int) -> Int -> Array Int -> Array Int
```

We pass only the first two arguments to `foldM`, leaving it hungry for one more, `Array Int`.
Partial `foldM` now has this signature:

```purescript
foldM' :: Array Int -> Array Int
```

which matches `sums`'s signature:

```purescript
sums :: Array Int -> Array Int
```

Let's get back to the folding function we pass to `foldM`.
Its signature might seem peculiar, because your typical reducer usually looks like this:

`(a -> b -> a)`

that is, it "merges" `b`s into `a` and then it's ready to be called again.

So we have a way to go from `a` to `m a`. But not from `m a` to `a`.
What the heck are we supposed to do when we have to pass `m a` into the "reducer" (or should I say "folder")?

Turns out, `foldM` does something behind the scenes making use of the fact that `m a` is a monad.

That's the source code for `foldM` as declared in the `Data.Array` module.

```purescript
-- | Perform a fold using a monadic step function.
foldM :: forall m a b. Monad m => (a -> b -> m a) -> a -> Array b -> m a
foldM f a = uncons' (\_ -> pure a) (\b bs -> f a b >>= \a' -> foldM f a' bs)
```

Let's see... 

`foldM` ends up "only" calling `uncons'`, which breaks an array into the first element `b` and the rest of the array `bs`, much like our JavaScript head-tail argument destructuring. Then it calls either the first function you pass it (if the array turned out to be empty) or the second one, passing it `b` and `bs`.

For `foldM` the former is `(\_ -> pure a)`

and the latter is `(\b bs -> f a b >>= \a' -> foldM f a' bs)`

WTH is `(\_ -> pure a)`? WTH is `a`?
Right, `a` is an `Int` and represents the initial value. So when we feed just the value, but no numbers, `pure a` is returned.
That would be the counterpart to our recursion ending condition.

`pure a` is function (see Applicatives LINK) that, roughly speaking, wraps a simple value into a monad.
In our case the monad is `Array`, don't we know how values get wrapped into arrays?!

Indeed:

```purescript
instance applicativeArray :: Applicative Array where
  pure x = [x]
```

Just create an array with the value as its only element, no big deal.

Hold on, that's precisely what we were doing in our JavaScript solution!
Good that someone thought about abstracting this little thing away, right?

Ooookay, maybe get another tea/coffee and some sweets for this one, we're finding out what

```purescript
(\b bs -> f a b >>= \a' -> foldM f a' bs)
```

does.


To recap, `b` and `bs` are the new `head` and `tail`.
Let's feed `b` together with `a`, the "sum" (scroll up to see where it was in JS code) argument into `f`

`f` is still that simple function we earlier passed into `foldM`: `\x y -> [x, x + y]`

For example, for initial zero sum and the first number `1` it will therefore return `[0, 1]`.
Those are the two computation paths we can take!

Then we stumble upon this weird `>>=`. It's called `bind` when it's not wearing infix.

Again, I recommend referring to this LINK great article for visualizing what is going on here.
Let's just say, `bind` knows how to extract stuff out of monadic containers and apply functions to said stuff, that return new monadic containers of the same shape.

The function we'd like to apply to `[0, 1]` is `\a' -> foldM f a' bs`. But _how_ is it applied?
If you leaf through the source code a bit, you'll discover that it `mapcat`s the array!

So, binding an array with a function means applying this (array-producing) function and concatenating the results!

_But that means that foldM is called twice, for 0 and 1, with the remaining numbers_.

That's where your pulse might heighten. Somewhat.

`foldM` will return another array, which is a concatenation of further arrays, all the way to the leaves.
So we recursively sumberge, happily binding along the way, then we get single-sum arrays, concat all the way back and get a flat array of all sums!

Our array-as-a-monad conveniently fits into the way `foldM`, well, folds.
It represents exactly what we want it to be for this problem: a structure of computation paths. That's a novel way to look at arrays, isn't it? When we want to fork our computation we create an array of two elements -- and get two paths, recursively followed!

Sit back and enjoy your tea/coffee and this tiny but delicious bit of newfound knowledge.


