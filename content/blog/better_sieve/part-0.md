+++
title = "Part 0: Background"
description = "Background information about sieving process"
date = 2023-03-25
updated = 2023-03-26
weight = 0

[taxonomies]
programming-language = ["python"]

[extra]
allow_comments = true
show_sibling_pages = true
+++
This post provides some background information about my plan to build a better sieve through Python generators.
It will go over Sieve of Eratosthenes, what I plan to do, and some ideas for going about implementing it.
As a baseline, one should have an understanding of prime numbers and be able to parse Python code.
<!-- more -->

## Starting with the Sieve of Eratosthenes
One simple way of generating prime numbers is to use the Sieve of Eratosthenes algorithm. 
The idea is to start with a list of all number up starting from 2 going up to some given number.
Each number in the list is assigned to true initially.
Then, loop through each number and mark off all multiples of that number (except itself).
Once you have gone through each of the numbers, go through the list again and keep only those numbers that are still true.
This now contains all the prime number up to some given number.

Here is a simple example in Python:
```python
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
        # For every offset (multiple) of the number, starting at n*2
        for offset in range(number*2, final_number, number):
            # Set that number to false (since one of it's factor is a prime not itself)
            number_check_list[offset] = False

    # Return a list that keeps only number whose value is true.
    return [number for number, is_prime in enumerate(number_check_list) if is_prime]
```
*Note: There are many different ways of making this more efficient such as starting with offset 2 for indices (there are no primes less than 2[^negative-primes-footnote]), skip processing number that are marked off (as their multiple are already known not to be primes from previous step), and stopping at the square root of the final number (since all composite numbers between the square root and final number will have been mark off at prior steps). This example tries to keep the algorithm simple without any deeper mathematical though required.*

## What do I want to do?

The main problem here is that the function has to hold all numbers before marking them off. 
This is especially bad for very large numbers as most numbers at that scale are composite.
In fact, we know that at minimum, half the numbers will not be prime (as they are even and thus divisible by 2).
This can be managed by using segmented variant of the sieve, which builds the list in pieces.

Another issue is the need to process all the numbers first before they can be use it.
However, Python provides generators to allow yielding values while they are being generated, allowing one to stream the numbers in and process them immediately.
By using generator, the function only needs to process data as it is generated, allowing for early stopping.

With this, I will endeavor to create a better kind of sieve that can mark off items while being generated.
The idea should be to iteratively create a generator whose value will always return a prime when calling next on it (at least up to a point).
The ideal metric to compare this solution to the Sieve of Eratosthenes is compactness (i.e. how much less memory is needed for the solution).

### Motivation

So, how did I come to think about this.
Well, I was working on the [Sieve exercise for Rust](https://exercism.org/tracks/rust/exercises/sieve) on [Exercism](https://exercism.org). 
In my [first iteration](https://exercism.org/tracks/rust/exercises/sieve/solutions/LightWithClocks), I didn't fully read the instructions correctly and implemented it as an iterator that calculates prime numbers by modding each current number with all previously found primes and checking if all are not 0 (evenly divisible).

When I went to redo the question for the second iteration, I used the `step_by` function to skip over multiples of a given value.
Since the step_by function itself is iterable, it gave me an idea to see if I could chain a bunch of `step_by` function together and yield primes that way.
If you see my second iteration for the solution, I did *not* actually implement that, but it did provide a seed for starting to think about such a solution.  

## Current Idea

Given some `BIG_NUMBER`, I want to find all the primes up to that big number and **yield** the values as they are found. 

```python
for prime in prime_generating_sieve(BIG_NUMBER):
    print(prime)

# prints out: 2, 3, 5, 7, 11, 13, 17, 19, ...
```

This can allow efficiently stopping before reaching the end:

```python
for prime in prime_generating_sieve(BIG_NUMBER):
    if prime < 10:
        print(prime)
    else:
        break

# prints out: 2, 3, 5, 7
```

It might also be possible to resume the iterator:
```python
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

```python
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


primes.append(next(range_0))
range_1 = (number for number in skip_every([1], range_0))

primes.append(next(range_1))
range_2 = (number for number in skip_every([2], range_0))

# ...
# ...

```
Given a range, I create a new range that skip over all multiples of 2 above 2 (first prime found).
Then, using that new range, I create another range that skips over all remaining multiples of 3 (next prime found).
Keep doing this until it we are guaranteed to have a generator that yields only primes (up to some number). 

In the next post, I document my attempt(s) to try and implement this.  

## Footnotes 
[^negative-primes-footnote]: While the assertion "there is no prime less than 2" is true for the common lay definition of prime, mathematically, there might a reason to consider negative version of primes as associates. [The PrimePages's FAQ on whether negative numbers can be prime](https://t5k.org/notes/faq/negative_primes.html) provides more information this.