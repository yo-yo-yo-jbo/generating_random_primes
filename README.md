# Generating random primes
[Last time](https://github.com/yo-yo-yo-jbo/rsa_math) I mentioned how (and why) the `RSA` algorithm works. The first part of the algorithm was to generate two large primes, which I took "for granted".  
However, it's unclear how one should do it since we do not have an *efficient* formula for generating primes (we do have [highly inefficient ways of generating primes](https://en.wikipedia.org/wiki/Formula_for_primes) though).  
In this blogpost I want to mention the most prevalent way to generate primes, using something called the [Miller-Rabin primality test](https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test).  
There are other ways of generating primes, but as far as I know Miller-Rabin is still considered the most popular one, albeit being probabilistic in nature. Let's go!

## Prime generation with primality tests
A naive (and not wrong) idea on how to generate a random prime number between 1 and some number `X` is the following idea:

1. Uniformly choose a random integer between `1` and `X` (how do we do that? That's for a future blogpost).
2. Check if the number is random. If it is - great! Otherwise, repeat step 1.

While that sounds silly, it's not a bad idea, assuming the primality test (step 2) has good time-complexity. Let's assume it does - what would the overall complexity be?  
Well, that depends on the expected number of primes in the range `1 - X`. That number is sometimes denoted as `π(X)` and has a good approximation - according to the [Prime number theorem](https://en.wikipedia.org/wiki/Prime_number_theorem) (which I won't prove now): `π(X)` is approximately `X / ln(X)`.  
That means that when choosing random number between `1` and `X`, the probability of hitting a prime is approximately `π(X) / X = (X / ln(X)) / X = 1 / ln(X)`.  
In terms of complexity, it's important to note our input is *the number of bits required to represent X rather than X itself*. If `X = 2^n` then `ln(X) = ln(2^n) = 2 ln(n)`.
