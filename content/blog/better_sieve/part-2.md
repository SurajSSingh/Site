+++
title = "Part 2: Taking a step back"
description = "Going over some ideas learned researching this problem."
date = 2023-05-06
# updated = 2023-05-07
weight = 2

[taxonomies]
programming-language = ["python"]

[extra]
allow_comments = true
show_sibling_pages = true
use_katex = true
+++

In this post, I will go over what I learned throughout my research related to my sieving attempts.
Think of this as an informal literature review for the sieve of Eratosthenes.
<!-- more -->

## What Does This Problem Ask

Before we continue, let's recall the goal of this problem.
I want to generate prime numbers using ranges, which successively yield values directly from them.
This should be done without requiring one to store non-prime numbers or using modulus to figure out if the number is non-prime.
One way to do this would be to send a range through a sieve that removes multiples of already known prime numbers.
From there, new sieves can be constructed using newly found prime numbers and eventually, all primes within some specified range will be found. 

### Is this just a wheel factorization?

No... Yes... Maybe it might be, I am not sure right now.
At a cursory glance, the pattern might have something to do with wheel factorization.
As a quick review of wheel factorization as it pertains to sieves, it is an optimization to the sieve of Eratosthenes where one can reduce the set of numbers to check by using a small group of numbers (also known as the **basis**) to quickly find numbers that cannot be prime. 

As an example, let's do the extremely simple basis of ${2, 3}$.
How might one skip any number divisible by $2$ or $3$, and do so in a cyclic way?
Separating that for $2$ and $3$, it would be to use every other number and skip every 3rd number respectively.

$div2$ = [1, `2`, 3, `4`, 5, `6`, 7, `8`, 9, `10`, 11, `12`, ...] 
$div3$ = [1, 2, `3`, 4, 5, `6`, 7, 8, `9`, 10, 11, `12`, ...] 

If one combines the two ranges, one get the following range:
$div2AndDiv3$ = [1, `2`, `3`, `4`, 5, `6`, 7, `8`, `9`, `10`, 11, `12`, ...]

With this, one can spot that every "6th" number is where the cycle repeats, which should track since it is the multiple of both 2 and 3.
Additionally, one might notice that 1, 5, 7, and 11 in the range are not divisible by 2 or 3, so those might be *candidates* for prime numbers.
What these numbers have in common is that they are co-prime to 6, that is, their only common divisor is 1.
From here, one only needs to check 1/3 of the numbers from the full range, as numbers divisible by $2$ or $3$ have already been removed.
More numbers in the basis can further reduce the number of checks needed, though with diminishing returns.

In this problem's case, the pattern might have something to do with wheel factorization, though it's not that clear from the outset.
However, besides that, I doubt there is anything else that is specific to the problem and wheel factorization, especially since wheel factorization is done at the beginning only once, whereas this problem focuses more creating new sieve (which could be wheels) from primes already found.
I wonder if there is something that might use wheel factorization more dynamically.

## Finding the pattern

Based on my current attempts to find the pattern, I will want to start by looking at the gap numbers.
More specifically, the numbers that remain between multiples of the prime I am working with.
I have a feeling that could have something to do with the wheel factorization.
If I can figure out that, I could use the [OEIS](https://oeis.org/)[^OEIS] to find similar gap numbers and maybe even a formula.
That in turn could guide me as to how I could generate patterns without needing to calculate by going through a copy of the range.

### The Code

The starting point to find the pattern is to first generate numbers and calculate the gaps.
The way I handled this was creating a simple function to count the numbers between two multiples of a given prime.
These numbers only exist if they were not multiples of any previous primes (though they may still be multiples of future primes).
Here is the function I came up with:

```python
def remaining_numbers_between_multiples(main_prime: int, prior_primes: list[int], final_number: int = 1000):
    # Setup
    counter = dict()
    start_number = main_prime
    current_count = 0

    for n in range(main_prime+1, final_number+1):
        # Not divisible by any prior primes
        if all(map(lambda p: n % p, prior_primes)):
            # Not divisible by the main prime
            if n % main_prime:
                current_count += 1
            else:
                # Save the count of numbers between start and n
                counter[(start_number, n)] = current_count
                # Reset count and start
                current_count = 0
                start_number = n

    return counter
```

Above, I have a dictionary called `counter` which maps the key of the start and end tuple to the count value.
Checking every number after the prime up to the final number, I skip over any numbers divisible by any prior primes, increment the counter if the number co-prime to the current prime, and finalize the counter range if the number is a multiple, which starts a new counter range.
Once that is done, I return the `counter`.

### Sequences found

As a check to verify that this actually works as intended, I verified that $2$, $3$, and $5$ follow their already found patterns of skip every other number for $2$, skip after every two numbers for $3$, and the cyclic skip pattern of 6 and 2 for $5$.
Here is the code:
{% accordion(summary="Validate 2, 3, and 5 pattern up to 100", render_markdown=true) %}
```python
print(remaining_numbers_between_multiples(2, [], 100))
print(remaining_numbers_between_multiples(3, [2], 100))
print(remaining_numbers_between_multiples(5, [2,3], 100))
```
Print out:
```
{(2, 4): 1, (4, 6): 1, (6, 8): 1, (8, 10): 1, (10, 12): 1, (12, 14): 1, (14, 16): 1, (16, 18): 1, (18, 20): 1, (20, 22): 1, (22, 24): 1, (24, 26): 1, (26, 28): 1, (28, 30): 1, (30, 32): 1, (32, 34): 1, (34, 36): 1, (36, 38): 1, (38, 40): 1, (40, 42): 1, (42, 44): 1, (44, 46): 1, (46, 48): 1, (48, 50): 1, (50, 52): 1, (52, 54): 1, (54, 56): 1, (56, 58): 1, (58, 60): 1, (60, 62): 1, (62, 64): 1, (64, 66): 1, (66, 68): 1, (68, 70): 1, (70, 72): 1, (72, 74): 1, (74, 76): 1, (76, 78): 1, (78, 80): 1, (80, 82): 1, (82, 84): 1, (84, 86): 1, (86, 88): 1, (88, 90): 1, (90, 92): 1, (92, 94): 1, (94, 96): 1, (96, 98): 1, (98, 100): 1}
{(3, 9): 2, (9, 15): 2, (15, 21): 2, (21, 27): 2, (27, 33): 2, (33, 39): 2, (39, 45): 2, (45, 51): 2, (51, 57): 2, (57, 63): 2, (63, 69): 2, (69, 75): 2, (75, 81): 2, (81, 87): 2, (87, 93): 2, (93, 99): 2}
{(5, 25): 6, (25, 35): 2, (35, 55): 6, (55, 65): 2, (65, 85): 6, (85, 95): 2}
```
{% end %}

With that pattern verified, I then checked $7$ and $11$.
I made sure that the pattern repeats by having the final number be sufficiently large enough to show at least two runs through the cycle. 

{% accordion(summary="Remain Numbers Between Multiples of 7 up to 500", render_markdown=true) %}
```python
remaining_numbers_between_multiples(7, [2,3,5], 500)
```
Which outputs: 
```
{
    (7, 49): 11,
    (49, 77): 6,
    (77, 91): 3,
    (91, 119): 6,
    (119, 133): 3,
    (133, 161): 6,
    (161, 203): 11,
    (203, 217): 2,
    (217, 259): 11,
    (259, 287): 6,
    (287, 301): 3,
    (301, 329): 6,
    (329, 343): 3,
    (343, 371): 6,
    (371, 413): 11,
    (413, 427): 2,
    (427, 469): 11,
    (469, 497): 6
}
```
{% end %}

{% accordion(summary="Remain Numbers Between Multiples of 11 up to 3000", render_markdown=true) %}
```python
remaining_numbers_between_multiples(11, [2,3,5,7], 5000)
```
Which outputs: 
```
{
    (11, 121): 25,
    (121, 143): 4,
    (143, 187): 9,
    (187, 209): 4,
    (209, 253): 10,
    (253, 319): 14,
    (319, 341): 3,
    (341, 407): 15,
    (407, 451): 9,
    (451, 473): 4,
    (473, 517): 8,
    (517, 583): 14,
    (583, 649): 15,
    (649, 671): 4,
    (671, 737): 14,
    (737, 781): 9,
    (781, 803): 4,
    (803, 869): 14,
    (869, 913): 10,
    (913, 979): 13,
    (979, 1067): 19,
    (1067, 1111): 10,
    (1111, 1133): 4,
    (1133, 1177): 8,
    (1177, 1199): 4,
    (1199, 1243): 10,
    (1243, 1331): 19,
    (1331, 1397): 13,
    (1397, 1441): 10,
    (1441, 1507): 14,
    (1507, 1529): 4,
    (1529, 1573): 9,
    (1573, 1639): 14,
    (1639, 1661): 4,
    (1661, 1727): 15,
    (1727, 1793): 14,
    (1793, 1837): 8,
    (1837, 1859): 4,
    (1859, 1903): 9,
    (1903, 1969): 15,
    (1969, 1991): 3,
    (1991, 2057): 14,
    (2057, 2101): 10,
    (2101, 2123): 4,
    (2123, 2167): 9,
    (2167, 2189): 4,
    (2189, 2299): 25,
    (2299, 2321): 2,
    (2321, 2431): 25,
    (2431, 2453): 4,
    (2453, 2497): 9,
    (2497, 2519): 4,
    (2519, 2563): 10,
    (2563, 2629): 14,
    (2629, 2651): 3,
    (2651, 2717): 15,
    (2717, 2761): 9,
    (2761, 2783): 4,
    (2783, 2827): 8,
    (2827, 2893): 14,
    (2893, 2959): 15,
    (2959, 2981): 4,
    (2981, 3047): 14,
    (3047, 3091): 9,
    (3091, 3113): 4,
    (3113, 3179): 14,
    (3179, 3223): 10,
    (3223, 3289): 13,
    (3289, 3377): 19,
    (3377, 3421): 10,
    (3421, 3443): 4,
    (3443, 3487): 8,
    (3487, 3509): 4,
    (3509, 3553): 10,
    (3553, 3641): 19,
    (3641, 3707): 13,
    (3707, 3751): 10,
    (3751, 3817): 14,
    (3817, 3839): 4,
    (3839, 3883): 9,
    (3883, 3949): 14,
    (3949, 3971): 4,
    (3971, 4037): 15,
    (4037, 4103): 14,
    (4103, 4147): 8,
    (4147, 4169): 4,
    (4169, 4213): 9,
    (4213, 4279): 15,
    (4279, 4301): 3,
    (4301, 4367): 14,
    (4367, 4411): 10,
    (4411, 4433): 4,
    (4433, 4477): 9,
    (4477, 4499): 4,
    (4499, 4609): 25,
    (4609, 4631): 2,
    (4631, 4741): 25,
    (4741, 4763): 4,
    (4763, 4807): 9,
    (4807, 4829): 4,
    (4829, 4873): 10,
    (4873, 4939): 14,
    (4939, 4961): 3
}
```
{% end %}

During this step, I also made a prediction for $13$ and $17$, though, those required much larger final numbers to finally get two cycle runs.
These are a very long sequences, so I've compressed it to only include the list of number through first cycle without showing multiples.
Below is the GitHub Gist of my predictions for 7, 11, 13, and 17:
{{gist(url="https://gist.github.com/SurajSSingh/f93e116273e2de0b622ca3c4ef286e06")}}

With this, let's try to find these sequences.

### Sequences on the OEIS

Heading over to the OEIS, let's type these sequences in (or at least most of it).
Getting the first sequence (prediction for 7), I got one entry found: [A236175](https://oeis.org/A236175).
This represents "Prime gap pattern of compacting Eratosthenes sieve for prime(4) = 7".
So now I know what this is called: "the prime gap pattern compacting up to a specific prime". 

This entry also links to similar patterns for [prime(5) = 11](https://oeis.org/A236176), [prime(6) = 13](https://oeis.org/A236177), [prime(7) = 17](https://oeis.org/A236178), [prime(8) = 19](https://oeis.org/A236179), and [prime(9) = 23](https://oeis.org/A236180).
These sequences match the predictions made for 11, 13, and 17; and so I would say finding the pattern was a success.

While there is a formula for $ prime(4) = 7$, there is no simple calculation for it (i.e. not requiring calculating the primes first) nor does it generalize up to the other primes.
As an alternative, I wanted to see if there was a way to calculate it based on prior sequences (i.e. 2 → 3 → 5).
I first look to see how the sequence cycle length grows for each next prime.

Using the length of the sequences found, I found the entry [A005867](https://oeis.org/A005867), which does not have a specific name, but I will refer to this as the "prime gap pattern length" sequence.
This sequence starts with $[1, 1, 2, 8, 48, 480, 5760, ...]$ and is a fast-growing sequence.
Calling number `len` on my sequences for the first 7 primes also get those numbers, which is great.
What's not great is how big those numbers are, as that means I need to have a giant list of numbers to find a pattern, which negates the efficiency of generating the number.

Given this revelation, I'm probably going to need to rethink how I generate the pattern, especially since I can't just store all the numbers when I want to create the skip pattern.

## Some other sieves
On the other side of my research is looking for similar sieves to what I am trying to achieve.
At a minimum, there appears to be 5 other sieves that generate primes, based on the Wikipedia page for the [sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes).
These are:
1. Wheel Factorization: Covered in the [prior section](#is-this-just-a-wheel-factorization) of this post.
2. Sieve of Atkin
3. Sieve of Sundaram
4. Euler's Sieve: Found [towards the bottom of the Sieve of Eratosthenes article](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes#Euler's_sieve)
5. Sieve of Pritchard

I will go over each of them and determine how closely they map to this problem.

### Sieve of Atkin
*Discovered by A. O. L. Atkin and Daniel J. Bernstein in 2003.*

The sieve of Atkin is a variant of the sieve of Eratosthenes that attempts to optimize it by checking that candidate primes are: 
1. not divisible by 2, 3, or 5; 
2. have an odd number of solutions to one of three specific equations based on its remainder modulo 60; 
3. and is square-free, that being numbers that are not divisible by any square number other than 1. 

While theoretically better, the sieve of Atkin only really achieves an advantage when the number of primes to calculate is very high.
For this problem, the primary idea of note is the idea of square-free numbers.
My attempts did not account for determining if a number was a multiple of a square, which let slip through some numbers that should not have been there.

For more detail on this sieve, see the [original paper by Atkin and Bernstein](https://www.ams.org/journals/mcom/2004-73-246/S0025-5718-03-01501-1/S0025-5718-03-01501-1.pdf), [Sieve Of Atkin post on Fylux Things](https://fylux.github.io/2017/03/16/Sieve-Of-Atkin/) ,and [this Medium post on Sieve of Atkin](https://medium.com/smucs/sieve-of-atkin-the-theoretical-optimization-of-prime-number-generation-e47107d61e28) [by Clay Wrentz](https://medium.com/@cwrentz).

### Sieve of Sundaram
*Discovered by S.P. Sundaram in 1934.*

The sieve of Sundaram is another variation on the sieve of Eratosthenes that has a different kind of elimination pattern.
Given a range from $1$ to $N$, $N$ being the final number, remove all numbers of the form $ a + b + 2ab $ where $ a $ and $ b $ are natural numbers with $ a \le b $.
Once all numbers of that form up to $ N $ are removed, multiply each remaining number by $2$ and add $1$, giving all the odd prime numbers.
Add on $2$ to the beginning of the list and one has the prime numbers up to $N$.
Unlike the sieve of Atkin, the sieve of Sundaram works better than the sieve of Eratosthenes when calculating a smaller range of primes, though become slower for higher range.

For further detail on this sieve, see [this article on Sundaram's Sieve by Julian Havil](https://plus.maths.org/content/sundarams-sieve) and [the Art of Problem Solving page on this sieve](https://artofproblemsolving.com/wiki/index.php/Sieve_of_Sundaram).

### Sieve of Pritchard and Euler's Sieve
*Sieve of Pritchard discovered by Paul Pritchard in 1979.*  
*Euler's Sieve originally used for Euler's proof of the Zeta Product formula, rediscovered by Gries & Misra in 1978.*

We now come to the final two sieves, and I've saved the best for last.
These two sieves are the closest to my idea of how the sieve is to work and how they work are quite interesting.
I'll be giving them their own separate post to go into more details about how the algorithm works, however, in this post, I will lay out the general idea of the sieves.

Let's start with the sieve of Pritchard, also known as *(**dynamic**) wheel sieve* (this seems familiar).
The idea is to generate prime numbers by constructing larger wheels at each step in the process to help skip over non-primes that have already been considered. 
A good analogy for this comes from the geometric view of this algorithm.

Create a wheel (`w1`) that has some `marks` and a `circumference` and let `p` be the current "prime" (prime in quotes since this does start at $1$).
At the start, `p` is set to 1, and the wheel's `circumference` is 1 and the only number in `marks` is at position 1.
Then, one needs to generate the next wheel with this process:
1.  "unroll" the wheel to find the next value `p`,
    * Note: the first two iterations unroll the wheel twice. This is because the wheel circumference is the same as `p`, and without doing it twice, would only produce values up to `p` (but not after).
2. construct a new wheel (`w2`) whose circumference is `p` times `w1`'s circumference,
3. roll the wheel `w1` inside `w2`, creating a new mark for `w2` at the position where a mark in `w1` touch `w2`,
4. finally, expand the wheel `w1` (i.e. multiply it) by `p` and remove any marks in common between `w1` and `w2`.

After this, one just needs to keep constructing new larger wheels until reaching the ending number $N$ (or more optimally, the square root of $N$).

The Euler's sieve is a modification of the sieve of Eratosthenes which works to only mark off numbers onces.
The way this is accomplished is by multiplying whatever the currently first prime is with every number currently in the list (i.e. not marked off) and then removing those numbers in the list, thus removing every multiple of the prime (including itself).
If one keeps track of all the first numbers, this gives one all the primes.
The sieve is effectively the fully working and correct version of my pattern generating skip list, as it includes all multiples for a given prime during each step.

These two sieves actually relate to each other in a specific way: when the final wheel is constructed for the sieve of Pritchard, all remaining operations are equivalent to running Euler's sieve.
Why is that?
That leads to my Eureka moment with this problem.

## My Eureka moment

Consider one wheel of the sieve of Pritchard.
One knows by wheel factorization that the `marks` should only be numbers that are co-prime to the `circumference` of the wheel.
Recall that the `circumference` of the wheel is the product of all the primes found until `p`.
If one rolls the wheel along the number line starting at 1, this will get all numbers co-prime to every prime before `p`.
Now, what happens if one were to multiply that wheel by `p` and it rolled along the number line.
This gives one every multiple of `p` that was not removed in the prior wheels.
If for each `p`, the wheel rolls the number line, this will remove all multiple of `p`.
Doing this is effectively the same as each step of the Euler's sieve in which `p` is the first number.

This correspondence between the wheel and the number line finally illustrates how the prime gap pattern is created and why it cycles.
Let's do $5$ as an example.
The cyclic gap pattern is: $[6,2]$.
One might wonder why it is those numbers.
Let's take a look at the wheel for $5$:
$[1,7,11,13,14,17,19,23,29]$.
If one adds back $5$ and $25$ (found by multiplying the previous wheel $3$ by $5$: $ [5•1, 5•5] $), one gets: $[1,\textbf{5},7,11,13,17,19,23,\textbf{25},29]$. 
Counting the numbers between them (assuming a cyclic list), one will see that it follows the $[6,2]$ patterns (technically $[1,6,1]$, but since it is cyclic, the first and last numbers are combine).

With this understanding, I have a better idea of how the skip pattern is created and why those numbers are kept in the range.

## Ideas for Next Attempts

So with all that I have learned, what's the next step?
First, I will likely use the sieve of Pritchard and Euler sieve as the new baseline, as those two sieve are the closest to what the sieve I have in mind.
I will also need to update how the pattern is found and stored, since I can't really have a giant list for the skip pattern (otherwise, why not just use the sieve of Eratosthenes).
As a possible idea, I could see if there is a way to dynamically generate the wheel through a different pattern while it is used to dynamically skip the range.

## Footnotes
[^OEIS]: **O**n-Line **E**ncyclopedia of **I**nteger **S**equences, a useful tool to see if some sequence of numbers have already been cataloged, can be used to find formulas or similar sequences.