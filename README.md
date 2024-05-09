# Generating random primes
[Last time](https://github.com/yo-yo-yo-jbo/rsa_math) I mentioned how (and why) the `RSA` algorithm works. The first part of the algorithm was to generate two large primes, which I took "for granted".  
However, it's unclear how one should do it since we do not have an *efficient* formula for generating primes (we do have [highly inefficient ways of generating primes](https://en.wikipedia.org/wiki/Formula_for_primes) though).  
In this blogpost I want to mention the most prevalent way to generate primes, using something called the [Miller-Rabin primality test](https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test).  
There are other ways of generating primes, but as far as I know Miller-Rabin is still considered the most popular one, albeit being probabilistic in nature. Let's go!

