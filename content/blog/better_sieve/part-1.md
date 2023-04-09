+++
title = "Part 1: First Attempts"
description = "A walkthrough of my idea and some attempts to implement it."
date = 2023-04-09
weight = 1

[taxonomies]
programming-language = ["python"]

[extra]
allow_comments = true
show_sibling_pages = true
use_katex = true
+++
In this post, I will go over some of my attempts to implement a better version of the Sieve of Eratosthenes.
<!-- more -->
If you would like to read up on the background information for this topic, please see [Part 0](@/blog/better_sieve/part-0.md).

<details>
<summary>TL;DR Summary</summary>
I have an idea to take a range generator and try to yield only prime numbers by figuring out a pattern to skip each multiple of the prime. The hard part is to somehow skip when previous skips are already applied (for example, when skipping multiples of 3, we don't need to consider 6, since we skipped it at 2). I then document 2 attempts at a solution, each of which works for small numbers, but at the mid 1000s.
</details>


## Simple Generator Version
To set a baseline, I created a simple generator version adapted from my [first iteration of the Sieve exercise for Rust](https://exercism.org/tracks/rust/exercises/sieve) on Exercism.


```python, linenos
def sieve_generator(end: int = 100):
    # List of primes currently found
    currently_found_primes = []

    # For every number from 2 up to the end number
    for number in range(2, end):

        # Note: (number % prime) when converted to boolean
        #       means the same as (number % prime) != 0
        
        # If for every prime in currently found primes, number % prime != 0...
        if all((number % prime for prime in currently_found_primes)):
            # Append it to currently found primes list,
            currently_found_primes.append(number)
            # and yield the number.
            yield number

    # Once finished, return the the list of found primes.
    # As this is a generator, this list is the value assigned
    # to the StopIteration exception
    return currently_found_primes
```

The goal for my attempts is to be as efficient and accurate as this.

*Note 1: This is meant to be the minimal standard version. Improvements can easily be made to this function.*  
*Note 2: This is a generator that also returns a value. One can either take primes as they are generated or wait until the StopIteration exception is raised to get the fully formed list.*

## Idea Summary
The main idea, as described in [part 0](@/blog/better_sieve/part-0.md#current-idea), is to take a range object and refine it with a pattern of skips to get all prime numbers out.
Both Python and Rust have lazy ranges, meaning the values inside the range are not held in memory, rather they are only produced when they are requested upon.
Ranges can be modified with skip and step-by functions to automatically jump over certain values.
All of this combined could help build a better generator function, as it would only need to hold the current range it is working on and yield from there.

## Setup

### Imports

```python, linenos
from collections import Counter, deque
from itertools import count, takewhile, tee
from more_itertools import peekable
```
1. [collections](https://docs.python.org/3/library/collections.html) from the standard library:
    * [`Counter`](https://docs.python.org/3/library/collections.html#collections.Counter): Used to hold a count of numbers.
    * [`deque`](https://docs.python.org/3/library/collections.html#collections.deque): Used for a queue data structure. 
2. [itertools](https://docs.python.org/3/library/itertools.html) from the standard library:
    * [`count`](https://docs.python.org/3/library/itertools.html#itertools.count): Used to emulate an infinite `range()`.
    * [`takewhile`](https://docs.python.org/3/library/itertools.html#itertools.takewhile): Keep taking from an iterator while some condition is true.
    * [`tee`](https://docs.python.org/3/library/itertools.html#itertools.tee): Used to make a copy of an iterator.
3. [more_itertools library](https://github.com/more-itertools/more-itertools):
    * [`peekable`](https://more-itertools.readthedocs.io/en/stable/api.html#more_itertools.peekable): Allow one item look-ahead for a generator/iterator.

### Helper classes and functions
Below are a set of helper functions:
A function that takes a generator and returns the first value and the remaining generator. Think of this as a combined `head` function and `tail` function starting at the first item from other languages like Haskell and R. I will use this to retrieve the first value while also skipping it in the generator (acts like `skip()`).
```python, linenos
def generator_take_first(gen):
    try:
        return next(gen), gen
    except StopIteration:
        return None, gen
```
2. A function that takes a generator and a list of how many indices to keeps, and then yields that many values, skipping the next value when the pattern changes. This match with the `keeper()` function used above.
```python, linenos
def generator_keep_then_skip_once(gen, keep_pattern = None):
    # Keep index when pattern is a list
    keep_index = None
    # If pattern is None or empty list, then keep everything
    if keep_pattern is None or keep_pattern == []:
        keep = 0
    elif isinstance(keep_pattern, int) and keep_pattern >= 0:
        keep = keep_pattern
    elif isinstance(keep_pattern, list) and len(keep_pattern) > 0:
        keep = keep_pattern
        keep_index = 0
    else:
        raise ValueError(f"Expected a None, int, or list of ints for keep_pattern: got {keep_pattern}")
    
    while True:
        try: 
            keep_by = keep if keep_index is None else keep[keep_index]
            # Keep these value
            for _ in range(keep_by):
                yield next(gen)
            # Skip this one
            next(gen)

            # Move to next item in pattern if pattern is a list
            # Cycle back around when it reaches the end
            if keep_index is not None:
                keep_index = (keep_index + 1) % len(keep)
            
        except StopIteration:
            break
```
3. A function that takes a list and tries to find the shortest repeating pattern length. This is able to also handle partially defined patterns, for example: given $[2,4,5,2,4,5,2]$, it will determine that a length of 3 is enough to repeat, even though the last "pattern" is not fully defined. This does create an issue of how do we know if the pattern is correct (there might not be a pattern). This function does not address this, merely its goal is to find the shortest possible which can satisfy if we assumed it repeated.
```python, linenos
def shortest_periodic_pattern(list_with_pattern: list[int]) -> int:
    max_pattern_len = len(list_with_pattern)
    for length in range(1, max_pattern_len):
        pattern = list_with_pattern[:length]
        chunks = [list_with_pattern[i:i+length] for i in range(0, max_pattern_len, length)]
        for chunk in chunks:
            # Check for any failing chunks
            if chunk != pattern[:len(chunk)]:
                break
        else: # no-break - found length
            return length
    # Default length is full length
    return max_pattern_len
```

## Attempts

### Attempt 1

Below is my first attempt at the problem, which successfully generates primes up to 1330.
I broke up the code into specific sections, but you can find the full code at the end of the section.

The overall steps are as follows:
1. Get the current prime to sieve the current range generator.
2. Create a list of multiples of the current prime that are still in the range.
3. Use that list of multiples to find a pattern for how many numbers to keep between each multiple.
4. Update the range generator to follow the keep pattern found.
5. Start again from step 1 if we have not reached the end.

#### Data Setup
Before the loop, I set up three variables to help out with the process:
* `already_sieved_primes`: A list of known primes that have already been used.
* `helping_sieve_primes`: A list of prime numbers that still need to be looked at and have their multiples removed.
* `current_running_range`: The **dynamic** range object that will change as it follows through the loop.
```python, linenos,linenostart=2
# Primes that have already been processed and whose multiples can never appear in the range
already_sieved_primes = []

# Prime numbers that are between p and p^2.
#   To help find the keep pattern
helping_sieve_primes = deque([2])

# The range that is running
#   Starts with unbounded range: 3... step by 1 
current_running_range = count(3,1)
```
The already sieved primes list is there for building the final list to return. 
If no return was needed, I could just remove that variable, as it serves no other purpose as of now.
On the other hand, the helping sieve primes list and current running range variables are central to the algorithm.
The helpers inside the list will provide all the primes still needing to go through the loop and will be used to help create the "in-between" items to figure out the pattern for keeping values.  
The current range keeps track of all the applied keep functions to make sure that it is producing the correct value.

#### Inside the Loop
In the main loop (currently implemented as a `while True`), we can go through the process of refining the sieve.

The first thing to do is to figure out the prime number to sieve out (from here, referred to as `p`).
In this case, we'll just take out the first prime number from the previous step(s) that still needs to be processed (in the helper list).
The currently running range can also be made peekable to allow looking at one next step. 
```python, linenos, linenostart=15
# The current prime to sieve (ref as "p")
sieving_prime = helping_sieve_primes.popleft()

# Make current range with one look ahead
current_running_range = peekable(current_running_range)
```

With the prime number to sieve determined, we can get all the numbers in the range up to the prime squared (`p^2`), as they are guaranteed to be prime (as `p^2` is the smallest current composite number in the range). 
This will do two things: 1) provide more numbers to help sieve out more multiple of the current prime `p`, and 2) move the range up to `p^2` and thus reduce the numbers need to "check".
```python, linenos, linenostart=15
# While the next number is smaller than p^2
while current_running_range.peek() < sieving_prime**2:
    # Add next value in range to helping sieve
    helping_sieve_primes.append(next(current_running_range))
```

We will then create a list of multiples to remove from the range, using `p` and the primes from the helper list.
It is not yet known if these numbers will be enough to determine the pattern, but I will work with the assumption that it will be.
We also need a copy of the current range and a counter.
The copy is to allow getting numbers with the range without actually moving the range.
The counter is to count the numbers that are in between each adjacent multiple.  
```python, linenos, linenostart=15
# Create a list of multiples divisible by sieving prime not already removed from before
between_list = [sieving_prime, sieving_prime ** 2] + [sieving_prime * p for p in helping_sieve_primes]
# Create a copy of the current range 
collapsable_range, current_running_range = tee(current_running_range, 2)
# Make the copied range peekable
collapsable_range = peekable(collapsable_range)
# Create a counter to count how many numbers are between two multiples of p
between_counter = Counter()
```

Now, let's count the numbers between each adjacent set of two multiples and use that to update the counter.
```python, linenos, linenostart=15
# For each start and end point (a number in between list and the next number after)
for start, end in zip(between_list, between_list[1:]):
    # Count how many primes from helping primes list are in between start and end
    for helper in helping_sieve_primes:
        if start < helper < end:
            between_counter[(start, end)] += 1

    # Skip the number if it is the start or end 
    peek_value = collapsable_range.peek()
    if peek_value == start or peek_value == end:
        next(collapsable_range)
        peek_value = collapsable_range.peek()

    # Count how many items that are between start and end inside the copied range
    while start < peek_value < end:
        next(collapsable_range)
        between_counter[(start, end)] += 1
        peek_value = collapsable_range.peek()
```

We can then use the values in the counter to create a list of numbers to keep that follow a specific pattern, using the `shortest_periodic_pattern` function to get the most compact pattern.
```python, linenos, linenostart=15
# Skip first in list because already processed
keep_list = [between_counter[key] for key in sorted(between_counter.keys())[1:] ]

# Find the pattern
pattern_length = shortest_periodic_pattern(keep_list)
pattern = keep_list[:pattern_length] if pattern_length > 1 else keep_list
```

Then we can start finishing up.
We can add `p` into the list of already sieved primes and update the max prime with the latest number in the helper list.
If the max prime is bigger than or equal to the final number to calculate, we are done with the loop and so we stop.
```python, linenos, linenostart=15
# Finished with prime so move it to already sieved
already_sieved_primes.append(sieving_prime)

# Get the max prime
max_prime = helping_sieve_primes[-1]

# Check that we have produced enough primes
if max_prime >= final_number:
    break
```
Otherwise, moving along, we can update the currently running range by removing the first value (should be `p^2`) and running the keep-skip function with the pattern.
```python, linenos, linenostart=15
# Remove square of prime for the current range
square_of_prime, current_running_range = generator_take_first(current_running_range)

# Verify that we have removed the prime squared as the first item
assert square_of_prime == sieving_prime ** 2 

# Produce the next range
current_running_range = generator_keep_then_skip_once(current_running_range, pattern)
```

#### Return
Once the loop is finished, return all the primes that have been already sieved as well as the number of primes still in the queue to be processed (up to the final number).
```python, linenos, linenostart=15
return already_sieved_primes + [p for p in helping_sieve_primes if p <= final_number]
```

#### Full Code
{% accordion(summary="Attempt 1 Full Code", render_markdown=true) %}
```python, linenos
# ALMOST WORKING - works until 1331
def my_new_sieve(final_number:int = 1330) -> list[int]:
    # Primes that have already been processed and whose multiples can never appear in the range
    already_sieved_primes = []

    # Prime numbers that are between p and p^2.
    #   To help find the keep pattern
    helping_sieve_primes = deque([2])
    
    # The range that is running
    #   Starts with unbounded range: 3... step by 1 
    current_running_range = count(3,1)

    # Run forever (allow infinite range)
    while True:
        # The current prime to sieve (ref as "p")
        sieving_prime = helping_sieve_primes.popleft()

        # Make current range with one look ahead
        current_running_range = peekable(current_running_range)
        
        # While the next number is smaller than p^2
        while current_running_range.peek() < sieving_prime**2:
            # Add next value in range to helping sieve
            helping_sieve_primes.append(next(current_running_range))

        # Create a list of multiples divisible by sieving prime not already removed from before
        between_list = [sieving_prime, sieving_prime ** 2] + [sieving_prime * p for p in helping_sieve_primes]
        # Create a copy of the current range 
        collapsable_range, current_running_range = tee(current_running_range, 2)
        # Make the copied range peekable
        collapsable_range = peekable(collapsable_range)
        # Create a counter to count how many numbers are between two multiples of p
        between_counter = Counter()

        # For each start and end point (a number in between list and the next number after)
        for start, end in zip(between_list, between_list[1:]):
            # Count how many primes from helping primes list are in between start and end
            for helper in helping_sieve_primes:
                if start < helper < end:
                    between_counter[(start, end)] += 1

            # Skip the number if it is the start or end 
            peek_value = collapsable_range.peek()
            if peek_value == start or peek_value == end:
                next(collapsable_range)
                peek_value = collapsable_range.peek()

            # Count how many items that are between start and end inside the copied range
            while start < peek_value < end:
                next(collapsable_range)
                between_counter[(start, end)] += 1
                peek_value = collapsable_range.peek()



        # Skip first in list because already processed
        keep_list = [between_counter[key] for key in sorted(between_counter.keys())[1:] ]

        # Find the pattern
        pattern_length = shortest_periodic_pattern(keep_list)
        pattern = keep_list[:pattern_length] if pattern_length > 1 else keep_list
        
        # Finished with prime so move it to already sieved
        already_sieved_primes.append(sieving_prime)
        
        # Get the max prime
        max_prime = helping_sieve_primes[-1]
        
        # Check that we have produced enough primes
        if max_prime >= final_number:
            break
        
        # Remove square of prime from the current range
        square_of_prime, current_running_range = generator_take_first(current_running_range)

       # Verify that we have reached the prime squared
        assert square_of_prime == sieving_prime ** 2 

        # Produce the next range
        current_running_range = generator_keep_then_skip_once(current_running_range, pattern)
        

    return already_sieved_primes + [p for p in helping_sieve_primes if p <= final_number]
```
{% end %}

#### Analysis
For my first attempt, the idea seems to be simple enough to follow. 
Have a range, gather some known primes, use that to generate a pattern for which numbers to keep, update the range, rinse and repeat.
Because of this simplicity, it might not be surprising that there are issues with the function.

The main issue I found is that while it works well for small numbers, it fails when it reaches 1331 (which is 11^3).
When running this, the function allows 1331 to be kept as a prime when it is not.
I appear to be missing the repeating pattern for when the range reaches 11.
There likely is an issue with developing the pattern for 11, either that it is incomplete or there is no pattern for 11.

Besides that, other improvements I could consider would be removing some unnecessary calculations. 
For example, start with the "evens" already sieved out and have 2 already processed.
Maybe also do the same for multiples of 3, as that is also an easy one to process.
Another idea might be to include more known primes in the helper list, say by getting all the additional primes between the first and second composite numbers, which could help with finding the pattern.

### Attempt 2

My second attempt uses the framework of the previous attempt and tries to improve it in certain ways. 
I will only go over the changes to the code, all others should be the same as the previous attempt.
See the full attempt towards the end of this section for the full code.

#### Update Setup
For the set-up before the loop, I chose to start from 3 rather than 2 to help with processing the range and remove one level of the keep function.
This means adding a check for the special case of the final number less than 2 (since 2 will have been processed once we reach the loop).
With that, 3 moves into the helping sieve list (in preparation for being the next prime to process) and the new range starts from 5 (one above the square of 2) and step up by 2 (skips even numbers).

One other code change I apply is calculating `max_prime` before the loop.
Since 2 is already processed, `max_prime` can be reasonably defined as 3 at the start (the last number in the primes to be processed).
This will help remove the need to add and conditional check inside the while loop (i.e. max prime check can directly be the condition for the while). 
```python, linenos, linenostart=2
# Special case: There are no primes less than 2
if final_number < 2:
    return []

# Primes that have already been processed and whose multiples can never appear in the range
already_sieved_primes = [2]

# Prime numbers that are between the current prime and its square.
#   To help find the keep pattern
helping_sieve_primes = deque([3])

# The range that is running
#   Starts the unbounded range with 5 and step by 2: [5,7,9,...)
current_running_range = count(5,2)

max_prime = helping_sieve_primes[-1]
```

#### Update Loop
With the max prime value being initialized before the loop, I can update the while loop to be `while max_prime < final_number` (i.e. conversion from a "do-while" to a regular "while" loop).

Then inside, I make a small update to the search for the number of primes.
In the attempt before, the loop would stop once it reaches the prime squared.
I still do this, but I skip over the prime value squared to run one more loop. 
```python, linenos, linenostart=15
# Count of how many numbers (all primes) are between p and p^2
while current_running_range.peek() <= sieving_prime**2:
    # Skip the first composite number and stop the loop
    if current_running_range.peek() == sieving_prime**2:
        next(current_running_range)
        break
    # Count the next prime helper number
    helping_sieve_primes.append(next(current_running_range))
```
Once the prime square is skipped, I can also take the number up to the multiplication of the current prime (`prime_0`) and the next prime (`prime_1`).
These numbers are also guaranteed to be prime because the next composite number is guaranteed to be the `prime_0` * `prime_1`, since it is the combination of the two smallest distinct primes[^remaining-composite-multiples-note].
This should provide more numbers to help with finding the pattern.
```python, linenos, linenostart=15
sieving_helper = helping_sieve_primes[0]
# While the next number is smaller than p*p+1, add the next number to helping list
while current_running_range.peek() < sieving_prime*sieving_helper::
    # Count the next prime helper number
    helping_sieve_primes.append(next(current_running_range))
```
Because of the expanded list of helping primes, I updated the between list to include the current prime cubed (as the new final multiple). 
```python, linenos, linenostart=15
between_list = sorted([sieving_prime, sieving_prime ** 2, sieving_prime**3] + [sieving_prime * p for p in helping_sieve_primes])
```
The for-loop after that finds the numbers are kept the same.
Once all the numbers are found and tallied up, I find the shortest repeating pattern like before, but with a small twist being that I have to move the beginning two numbers to the end.
This is because getting the prime before the current prime squared has already exhausted the range.
Similar for the primes up to current prime times next prime.
In order to keep the order correct, this means "rotating" the list to shift everything two units back and move the first two to the end.
```python, linenos, linenostart=15
# Find the pattern
pattern_length = shortest_periodic_pattern(keep_list)
pattern = deque(keep_list[:pattern_length] if pattern_length > 1 else keep_list)
pattern.rotate(-2)
pattern = list(pattern)
```
Once the pattern is determined, the current prime can be moved to the already processed list and the current range can be updated.
Since the range has removed all numbers till the multiplication of the two lowest primes, I had to update the assertion.
```python, linenos, linenostart=15
# Remove prime_0 * prime_1 from the current range        
mult_of_lowest_two_primes, current_running_range = generator_take_first(current_running_range)

# Verify that we have reached prime_0 * prime_1
assert mult_of_lowest_two_primes == sieving_prime * sieving_helper 
```
After the update, the while-loop should automatically stop once the number of primes reaches the final number (likely slightly faster since there are more numbers).

#### Full Code

{% accordion(summary="Attempt 2 Full Code", render_markdown=true) %}
```python, linenos
# ALMOST WORKING  - works until 1573
def my_new_sieve(final_number:int = 1573) -> list[int]:
    
    # Special case: There are no primes less than 2
    if final_number < 2:
        return []

    # Primes that have already been processed and whose multiples can never appear in the range
    already_sieved_primes = [2]

    # Prime numbers that are between p_0 (currently processing prime) and p_1 (prime just after)
    #   To help find the keep pattern
    helping_sieve_primes = deque([3])
    
    # The range that is running
    #   Starts the unbounded range with 5 and step by 2: [5,7,9,...)
    current_running_range = count(5,2)

    max_prime = helping_sieve_primes[-1]

    # while max prime is less than the final number 
    #   (meaning not all primes reached in range of final number)
    while max_prime < final_number:
        # The current prime to sieve (aliased as "p" in comments)
        sieving_prime = helping_sieve_primes.popleft()

        # Make current range with one look ahead
        current_running_range = peekable(current_running_range)
        
        # Count of how many numbers (all primes) are between p and p^2
        while current_running_range.peek() <= sieving_prime**2:
            # Skip the first composite number and stop the loop
            if current_running_range.peek() == sieving_prime**2:
                next(current_running_range)
                break
            # Count the next prime helper number
            helping_sieve_primes.append(next(current_running_range))
        
        sieving_helper = helping_sieve_primes[0]
        # While the next number is smaller than p*p+1, add the next number to helping list
        while current_running_range.peek() < sieving_prime*sieving_helper::
            # Count the next prime helper number
            helping_sieve_primes.append(next(current_running_range))
        
        # Create a list of multiples divisible by sieving prime not already removed from before
        between_list = sorted([sieving_prime, sieving_prime ** 2, sieving_prime**3] + [sieving_prime * p for p in helping_sieve_primes])
        # Create a copy of the current range 
        collapsable_range, current_running_range = tee(current_running_range, 2)
        # Make the copied range peekable
        collapsable_range = peekable(collapsable_range)
        # Create a counter to count how many numbers are between two multiples of p
        between_counter = Counter()

        # For each start and end point (a number in between list and the next number after)
        for start, end in zip(between_list, between_list[1:]):
            # Count how many primes from helping primes list are in between start and end
            for helper in helping_sieve_primes:
                if start < helper < end:
                    between_counter[(start, end)] += 1
 
            # Skip the number if it is the start or end 
            peek_value = collapsable_range.peek()
            if peek_value == start or peek_value == end:
                next(collapsable_range)
                peek_value = collapsable_range.peek()

            # Count how many items that are between start and end inside the copied range
            while start < peek_value < end:
                next(collapsable_range)
                between_counter[(start, end)] += 1
                peek_value = collapsable_range.peek()


        # Skip first in list because already processed
        keep_list = [between_counter[key] for key in sorted(between_counter.keys())]
       
        # Find the pattern
        pattern_length = shortest_periodic_pattern(keep_list)
        pattern = deque(keep_list[:pattern_length] if pattern_length > 1 else keep_list)
        pattern.rotate(-2)
        pattern = list(pattern)
        
        # Finished with prime so move it to already sieved
        already_sieved_primes.append(sieving_prime)
        
        # Get the max prime
        max_prime = helping_sieve_primes[-1]

        # Remove prime_0 * prime_1 from the current range        
        mult_of_lowest_two_primes, current_running_range = generator_take_first(current_running_range)
        
        # Verify that we have reached prime_0 * prime_1
        assert mult_of_lowest_two_primes == sieving_prime * sieving_helper 

        # Produce the next range
        current_running_range = generator_keep_then_skip_once(current_running_range, pattern)

    return already_sieved_primes + [p for p in helping_sieve_primes if p <= final_number]
```
{% end %}

#### Analysis
With this second attempt finished, I have done slightly better than my attempt 1.
Now, it fails at 1573 (11^2 * 13). 
I think I see the problem being just that I don't use the square as another thing to multiply the helper primes by, but this illustrates a more fundamental issue: I have no idea how the pattern is calculated.
While the idea to pre-process the range did help remove certain steps of the calculations and adding more helping primes helped increase the pattern sequence, neither did much for finding the actual pattern.

The big thing here is that I need to figure out how the pattern is derived.
Currently, I get it by making a copy of the range and assuming the repetition happens.
However, that might not be the case, so it might be time to look into the underlying math of this problem.

## Conclusion

So based on these results, did I succeed? 
While I didn't get to the result of actually getting the generator working, this did provide some good insight into how I might approach future attempts.
Namely, I am going to need to figure out how best to find the pattern for any given prime and how much information is needed to accomplish that task.

In the next post, I will take a step back and try to review some of the mathematical ideas present for this problem and review other kinds of solutions related to the Sieve of Eratosthenes.

## Footnotes
[^remaining-composite-multiples-note]: One might try to see if the third composite number would always be the current prime times the second next prime (which could give even more primes to help). The smallest counterexample is $5$: $5 * 11 = 55$ but $7^2 = 49$.    