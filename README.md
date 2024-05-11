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
That means that when choosing random number between `1` and `X`, the probability of hitting a prime is approximately:

`π(X) / X = (X / ln(X)) / X = 1 / ln(X)`.

In terms of complexity, it's important to note our input is *the number of bits required to represent X rather than X itself*.  
If `X = 2^n` then `ln(X) = ln(2^n) = n ln(2) = O(n)`, which means the chances of a "lucky draw" is in the order of `1/n`.  
That is encouraging! As I mentioned in a previous blogpost, `4096` bits is considered pretty safe (nowadays anyway!), so checking for primality around `4096` times is not too bad.  
Of course, this all depends on the primality test complexity. In order to be efficient, it has to also be some constant power of `ln(X)`, i.e. `(ln (X))^k` for some constant `k`.  
Naive primality tests such as checking if any number between `2` and `sqrt(x)` divide `x` are not good, since their complexity is linear in `x`, and `O(x) = O(2^n)`.  

## Primality check ideas
After we've established a naive primality test isn't a good idea, we need an alternative approach.  
One approach that is often used in math is finding certain qualities of properties of a certain set of numbers, that isn't true for other numbers.  
Well, what do we know primes, besides their definition? A few things come to mind, at least for me:
1. [Wilson's theorem](https://en.wikipedia.org/wiki/Wilson%27s_theorem): for a prime `n` the following is true: `(n-1)! = -1 (mod n)`, *and vice versa*. While proving it is not difficult, we will not do it today, nor will we use that since factorization is extremely computationally expansive (`O(n^n)` according to [Stirling's approximation](https://en.wikipedia.org/wiki/Stirling's_approximation)).
2. The fact that `Z*n` ([the multiplicative Group mod n](https://en.wikipedia.org/wiki/Multiplicative_group_of_integers_modulo_n)) has the size `n-1`. That's just a different way of saying that there all numbers in the range `2,3,...,n-1` are coprime to `n`, so it's kind of a repetition of the definition. However, saying that `Z*n` is a Group is quite powerful, and, in fact, we will use a certain quality that goes a bit beyond that today.
3. [Fermat's Little thorem](https://en.wikipedia.org/wiki/Fermat%27s_little_theorem) (not to be confused with [Fermat's Last theorem](https://en.wikipedia.org/wiki/Fermat's_Last_Theorem)!) which state that if `p` is a prime number then `a^p = a (mod p)` for every `a` that is coprime to `p`. Another way to formulate it that you might find in literature is `a^(p-1) = 1 (mod p)`, which is, of course, equivalent.

### Fermat's Little theorem
We'll start with Fermat's Little theorem, since it's quite powerful. It's obvious if we find an `a` that is coprime to `n` such that `a^(n-1)` is not `1 (mod n)` then obviously it's a definite proof that `n` is not prime. Well, what happens otherwise? Of course we couldn't check all values in the range due to the time complexity, so, in the spirit of probabilistic algorithms, we'd hope for a non-prime `n` there are many values of `a` that violate the equation in Fermat's Little theorem.  
Unfortunately, that's not the case, and, in fact, there are composite numbers (known as [Carmichael Numbers](https://mathworld.wolfram.com/CarmichaelNumber.html)) that *always* satisfy Fermat's Little theorem (for *every* `a`), even though they're composite.  
Our solution would be using Fermat's Little theorem, but also use another property of primes while calculating `a^(n-1) mod n`.

### Non-trivial roots of 1
I mentioned that a nice property of primes is that the [multiplicative Group mod n](https://en.wikipedia.org/wiki/Multiplicative_group_of_integers_modulo_n) contains all numbers between `2` and `n-1`.  
We could use that property in a nice way while calculating `a^(n-1) (mod n)`, by noticing that non-primes might have non-trivial roots of 1.  
What is a root of 1? It's just a number that satisfy `x^2 = 1 (mod n)`. If `n` is prime then there are only two such roots: `1` and `-1` (which is really `n-1`).  
For example, for `n=7` we get `6^2 = 36 = 1 (mod 7)`, and `1^2 = 1 (mod 7)`, but no other number satisfies that.  
However, for `n=8` we get `1^2 = 3^2 = 5^2 = 7^2 (mod 8)`, and therefore, `3` and `5` are non-trivial roots of 1.  
I will not explain exactly why that happens, but I will give a hint - under `mod 8` we have zero-divisors - `2` and `4`, which makes `3` and `5` "kind of behave" like 1, making them non-trivial roots of 1.  
With that in mind, let's see how we can efficiently use those two properties!

### Efficient modular exponentiation
Before we continue to our algorithm, let's remember we will need to compute `a^(n-1) (mod n)`, and we will have to do so efficiently.  
Naively, we multiply `a` by itself `n` times, and each time take the result `mod n`, which has a linear complexity in `n`, which is not very efficient. Can we do better?  
Well, we don't *really* need to multiply `a` by itself `n` times. For example, to calculate `a^8` we need to take `a^2` and then square it, and square that result, i.e. `a^8 = ((a^2)^2)^2`. That approach is excellent for powers of two, but what do we do with non-powers of two?  
Well, or number `n-1` can be represented as `n-1 = 2^s*r` where `r` is odd and `s` is some integer. We can calculate the powers of two very efficiently (takes `s` steps), so we only need to calculate `a^r (mod n)` efficiently.  
One of the coolest approach we could take is to look at the binary representation of `r` and multiply those as powers of 2. Let's see an example!  
Assuming we want `a^13` we could note `13` is `1101` in binary, and we note: `a^13 = a^8 * a^4 * a^1`. Of course, all of those are powers of 2 so they can be calculated quite efficiently.



