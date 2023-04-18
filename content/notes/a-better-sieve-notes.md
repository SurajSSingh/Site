+++
title = "Notes for Part 2 of \"A Better Sieve\""
description = "Some quick notes for things I will look at before writing Part 2"
date = 2023-04-17

[extra]
allow_comments = true
license = "CC BY 4.0"
+++

Below is a list of items to look at before writing part 2.
No other comment is given.

## OEIS Sequences
* [Prime gap pattern of compacting Eratosthenes sieve for prime(k=4) = 7](https://oeis.org/A236175) and similar
    * [k=5](https://oeis.org/A236176)
    * [k=6](https://oeis.org/A236177)
    * [k=7](https://oeis.org/A236178)
    * [k=8](https://oeis.org/A236179)
    * [k=9](https://oeis.org/A236180)
* [Sequence of gaps between deletions of multiples of 7 in step 4 of the sieve of Eratosthenes](https://oeis.org/A359632)
* [Differences between terms of compacting Eratosthenes sieve for prime(4) = 7](https://oeis.org/A236185)

## Other Sieves
* [Wheel Factorization](https://en.wikipedia.org/wiki/Wheel_factorization)
* [Sieve of Atkin](https://en.wikipedia.org/wiki/Sieve_of_Atkin)
* [Sieve of Sundaram](https://en.wikipedia.org/wiki/Sieve_of_Sundaram)
* **[Sieve of Pritchard](https://en.wikipedia.org/wiki/Sieve_of_Pritchard)**: *Important*, might be the sieve I am trying to make or very close to it

## Open Source Implementation
* [Primes](https://github.com/PlummersSoftwareLLC/Primes): Sieve of Eratosthenes implemented in many different programming languages and with different levels of optimization.
* [primesieve](https://github.com/kimwalisch/primesieve): C++ library for generating primes quickly, has binding in other languages such as Python and Rust.