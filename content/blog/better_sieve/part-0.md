+++
title = "Part 0: Background and Intro"
description = "Background information about sieving process and an introduction to the coding idea."
date = 2023-03-25
updated = 2023-03-29
weight = 0

[taxonomies]
programming-language = ["python"]

[extra]
allow_comments = true
show_sibling_pages = true
use_katex = true
+++

This post provides some background information about my plan to build a better sieve through Python generators.
It will go over the Sieve of Eratosthenes, what I plan to do, and some ideas for going about implementing it.
As a baseline, one should have an understanding of prime numbers and be able to understand Python code.
<!-- more -->

## Starting with the Sieve of Eratosthenes
One simple way of generating prime numbers is to use the Sieve of Eratosthenes algorithm. 
The idea is to start with a list of all numbers starting from 2 and going up to some given number.
Each number in the list is assigned to true initially.
Then, loop through each number and mark off all multiples of that number (except itself).
Once you have gone through each of the numbers, go through the list again and keep only those numbers that are still true.
This now contains all the prime numbers up to some given number, as any number that has factors beyond 1 and itself will be marked off during the first looping process.

Here is a simple example in Python:
```python, linenos
def sieve_of_eratosthenes(final_number: int = 100) -> list[int]:
    # Special case: No prime less than 2 -> empty list
    if final_number < 2:
        return []

    # Final number is +1 since Python is 0-indexed 
    final_number = final_number + 1

    # Create a number list with all positions from 0 up to final number set to True.
    # This assumes all numbers are prime, which will be refuted in the loop below.
    number_check_list = [True] * final_number
    
    # Known: 0 and 1 are not prime
    number_check_list[0] = False
    number_check_list[1] = False

    # For every number starting at 2 (inclusive) going to final number (exclusive)
    for number in range(2, final_number):
        # For every multiple of the number except itself up to the final number 
        # (i.e. n*2, n*3, n*4, ...)
        for multiple in range(number*2, final_number, number):
            # Set that number to false (since one of it's factor is this number)
            number_check_list[offset] = False

    # Return a list that keeps only number whose value is true.
    return [number for number, is_prime in enumerate(number_check_list) if is_prime]
```
*Note: There are many different ways of making this more efficient such as starting with offset 2 for indices (there are no primes less than 2[^negative-primes-footnote]), skipping numbers that are already marked off (as their multiple are already known not to be primes from the previous steps), and stopping at the square root of the final number (since all composite numbers between the square root and final number will have been marked off at prior steps). This example tries to keep the algorithm simple without any deeper mathematical proof required.*

## What do I want to do?

The main problem here is that the function has to hold all numbers before marking them off. 
This is especially bad for very large numbers as most numbers at that scale are composite.
We know that at minimum, half the numbers will not be prime (as they are even and thus divisible by 2).
This can be managed by using a segmented variant of the sieve, which builds the list in pieces.

Another issue is the need to process all the numbers first before they can be used.
However, Python provides generators to allow yielding values while they are being generated, allowing one to stream the numbers in and process them immediately.
By using generators, the function only needs to process data as it is generated, allowing for early stopping.

With this, I will endeavor to create a better kind of sieve that can mark off items while being generated.
The idea should be to iteratively create a generator whose value will always return a prime when calling next on it (at least up to a point).
The primary metric to compare this solution to the Sieve of Eratosthenes is compactness (i.e. how much less memory is needed for the solution).

### Motivation

So, how did I come to think about this idea?
Well, I was working on the [Sieve exercise for Rust](https://exercism.org/tracks/rust/exercises/sieve) on [Exercism](https://exercism.org). 
In my [first iteration](https://exercism.org/tracks/rust/exercises/sieve/solutions/LightWithClocks), I didn’t fully read the instructions correctly and implemented it as an iterator that calculates prime numbers by modding each current number with all previously found primes and checking if all are not 0 (evenly divisible).

When I went to redo the question for the second iteration, I used the `step_by` function to skip over multiples of a given value.
Since the step_by function itself is iterable, it gave me an idea to see if I could chain a bunch of `step_by` functions together and yield primes that way.
If you see my second iteration for the solution, I did *not* implement that, but it did provide a seed for starting to think about such a solution.  

## Current Idea

The idea for this new sieve is to start with a range of natural numbers stepping by 1 and somehow remove factors of a given prime by skipping over them to produce a next new range. At each successive step, the range is refined to only include primes and composites that don't hold any previous primes as factors. 

As an example, let's start with all the Natural numbers, which will be our first range (`range_0`). 
$$\mathbb{N} = range_0 = [0,1,2,3,4,5,6,7,8,9,10,\...]$$

To begin the algorithm, we will remove $0$ and $1$. 
This means skipping the first two numbers and then continuing with the remaining range, which will yield the second range (`range_1`).
$$range_0.skip(2) = range_1 = [2,3,4,5,6,7,8,9,10,11,12,\...]$$

Then, we start the algorithm proper.
In the beginning, we take the first number as prime and then somehow remove all its multiples.
In this case it's $2$ (which is the first prime) and to get rid of any multiple of $2$, we skip over them.
For $2$, we just need to skip every other number after $2$ (i.e. keep next, skip after). 
The result will look something like this (range_2).
$$range_1.skip(1).keeper(cycle([1])) = range_2 = [3,5,7,9,11,13,15,17,19,21\...]$$

2 Notes:
1. I assume that we already take the first value out before the `skip` function runs. Otherwise, we would lose the prime after we just calculated it in the range. This means the range just holds what remains to be calculated, not the entire prime list. This is fine, since our function is a generator, and will yield the first value out.
2. The `keeper` function works by taking a pattern of items to keep and skipping over when the keep pattern changes. This is why `cycle([1])` is needed, as we need to keep 1 number, then skip over the next, and then repeat this for the entire range.

Now we loop back to the beginning step. 
We take the first number as prime, which is $3$, and get rid of its multiples.
But, notice that some multiples of $3$ were removed in the previous step, namely all even multiples (divisible by $2$, the prior prime).
This means we need to skip all multiples of $3$ that are not multiples of $2$.

Fortunately, removing the evens did not change the number we need to skip by.
All multiples of $3$ are separated by 3 in `range_1` and also separated by 3 in `range_2`.
For example, every multiple 3 in the full range is separated by 3 (should be obvious, since it is a multiple of 3).
When we remove the even numbers, all multiples are now separated by 6 (e.g. 3 -> 9, 9 -> 15, 15 -> 21, etc.).
However, removing even numbers also removes half the numbers between the multiples, so 6 becomes 3.

Thus, we need just need to repeatedly keep every 2 numbers and skip once to produce the next range (`range_3`). 
$$range_2.skip(1).keeper(cycle([2])) = range_3 = [5,7,11,13,17,19,23,25,29,31,\...]$$

Let's do one more run through the loop (this one is a bit more interesting).
The next first number is $5$ and now we need to get rid of all multiples of $5$ not divisible by $2$ and $3$.
Let's consider what those numbers are: 
$$[5,25,35,55,65,85,95, \...]$$.
The numbers appear to follow a [**+20**, **+10**, **+20**, **+10**, ...] cyclic pattern.
Not only that, but the remaining numbers in-between also appear to follow a similar cyclic pattern: 
$$[\textbf{5},7,11,13,17,19,23,\textbf{25},29,31,\textbf{35},37,41,43,47,49,53,\textbf{55},59,61,\textbf{65},\...]$$.
This pattern is: [keep 6, skip 1, keep 2, skip 1] repeated forever (to my current knowledge).
This produces the final range we will look at `range_4`.
$$range_3.skip(1).keeper(cycle([6,2])) = range_4 = [7,11,13,17,19,23,29,31,37,41,43,47\...]$$

This calculation states to skip the first number ($5$), then keep the next 6 numbers (i.e. $7,11,13,17,19,23$), skip the next number ($25$), then keep 2 more numbers (i.e. $29, 31$), skip 1 ($35$), and then repeat the keep 6, skip 1, keep 2, skip 1 step.

We would keep running this until the largest prime up to some final number is found or after we stop yielding more value.

### Noticed Properties
One thing to notice is that during the processing of the range, all numbers between the first prime `p` and its square `p^2` are prime. 
This should make sense as any composite number with factors below `p` would have been removed in previous steps, and the first composite must be some multiple of `p` that does not share any factor in common with any primes below. 
The first number that satisfies that is `p^2`.
This can help improve the stopping condition, since if we know the final number to generate, we can stop processing once we have a prime at least as large as the final number. 

Another thing to notice is that skipping will not always be consistently the same number, as seen with `range_4` using $5$ to calculate. 
I don't know if there is a specific name in mathematics for this concept, but the idea is similar to a `step_by` function which can dynamically change.
To allow for this, I use a helper function to allow cyclically keeping a set of values and then skipping one next value.

As for the pattern, finding what list of keeps and skips satisfies the range mathematically is a bit outside my scope of knowledge.
Instead, one way to find the pattern would be to calculate how many numbers are in-between two (remaining) multiples and look for when this pattern of gaps repeats.
Although this requires copying the currently running generator to retrieve those values and waiting until the pattern repeats (which can be very wasteful), this is the current approach I took for my coding attempts. 
I hope to eventually use what I learn from the attempts to find the pattern either empirically with some function or mathematically with a formula.


### Example Code

Given some `BIG_NUMBER`, I want to find all the primes up to that big number and **yield** the values as they are found. 
The function for this should be a generator to be used like so:

```python, linenos
for prime in prime_generating_sieve(BIG_NUMBER):
    print(prime)

# prints out: 2, 3, 5, 7, 11, 13, 17, 19, ...
```

This can allow efficient stopping before reaching the end:

```python, linenos
for prime in prime_generating_sieve(BIG_NUMBER):
    if prime < 10:
        print(prime)
    else:
        break

# prints out: 2, 3, 5, 7
```

It can also be possible to resume the iterator at a later time:
```python, linenos
prime_generator = prime_generating_sieve(BIG_NUMBER)

# Previous prime needed as we need to call next on generator 
# before we can check the value, which consume it in the iterator.
# Might be better if we could peek the value before consuming it.
previous_prime = None

for prime in prime_generator:
    if prime < 10:
        print(prime)
    else:
        previous_prime = prime
        break
# prints out: 2, 3, 5, 7
# previous prime become 11

if previous_prime is not None:
    print(previous_prime)
    previous_prime = None

# ...
# Continue from where we left off

for next_prime in prime_generator:
    if prime < 20:
        print(prime)
    else:
        previous_prime = prime
        break
# prints out: 11, 13, 17, 19
# previous prime become 23
```

### A starting point

Here is my initial idea for where to start:

```python, linenos
primes = []
range_0 = iter(range(2, BIG_NUMBER))

# Skip every nth number in a cyclic list
def skip_every(skip_pattern, generator):
    if not isinstance(skip_pattern, list):
        raise TypeError("Skip pattern must be a list")
    
    # Skip nothing if skip pattern is empty
    if skip_pattern == []:
        skip_pattern = [0]

    skip_index = 0

    while True:
        try:
            yield next(generator)
            skip = skip_pattern[skip_index]
            for i in range(skip):
                next(generator)
            skip_index = (skip_index + 1) % skip_index
        except StopIteration:
            break

    for i in generator:
        yield i

# 2-multiples
primes.append(next(range_0))
range_1 = (number for number in skip_every([1], range_0))

# 3-multiples without 2
primes.append(next(range_1))
range_2 = (number for number in skip_every([2], range_1))

# 5-multiples without 2 or 3
primes.append(next(range_2))
range_3 = (number for number in skip_every([6,2], range_2))

# ...
# ...

```
Given a range, I create a new range that skips over all multiples of 2 above 2 (first prime found).
Then, using that new range, I create another range that skips over all remaining multiples of 3 (next prime found).
After that, another range is created that skips over all remaining multiples of 5 with the [6,2,6,2,...] pattern.
Keep doing this until we are guaranteed to have a generator that yields only primes (up to some number). 

In the next post, I document my attempt(s) to try and implement this as an actual generator, as well as describe the result. 

## Footnotes 
[^negative-primes-footnote]: While the assertion “there is no prime less than 2” is true for the common lay definition of prime, mathematically, there might a reason to consider a negative version of primes as primes themselves. [The PrimePages’ FAQ on whether negative numbers can be prime](https://t5k.org/notes/faq/negative_primes.html) provides more information on this.