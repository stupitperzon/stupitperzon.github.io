---
date: "2024-03-24"
tags: ["cryptography", "math", "cs"]
title: "GoFetch Attack Notes"
---

Disclaimer: I am not affiliated with the paper. Notes may be wrong.

```
> Be Apple
> Release new hardware-specific prefetching technique
> Call it DMP
> Stands for Data Memory-dependent Prefetchers
> Most heavily enabled on Apple M-series chips 
> Specific to Firestorm cores
> DMP prefetches memory that look like pointers
> It dereferences pointers so next time they're dereferenced faster
> They're faster because they're now preloaded in L1 or L2 caches
> Without DMP, the memory would reside in DRAM
> That'd double the CPU cycles for dereferencing
> This makes cryptography vulnerable to new side-channel attacks
> Cryptographers are mad since chosen plaintext attacks are back
> The right plaintext reveals whether intermediate state is/isn't in cache
> That reveals secret keys
```

## Cache implementation details

```
> Every processor has multiple cores
> Each core has an L1 cache
> L2 cache is shared by all cores
> L1, L2 are called cache levels
> Each of them contain cache sets
> Sets are indexed by page frames
> Each set contains cache lines
> Cache lines are weaved in all cache levels
> They're 64 bytes each for Firestorm
> L1 < L2 < DRAM in terms of size
> L1 > L2 > DRAM in terms of cost to make and speed
> L1, L2, DRAM is volatile memory
> They go away after restarts
```

## When is DMP triggered

```
> Triggered on single access to a memory location
> Access does not even mean dereference
> All pointers in the same cache line will be prefetched by DMP
> DMP does Translation Lookaside Buffer (TLB) lookup
> Pointers are virtual memory addresses after all
> There's limited DRAM
> So sometimes, pages need to be brought in from disk
> TLB converts virtual memory addresses to physical ones
> If there's a TLB miss, DMP inserts the pointer in TLB
> Need to account for edge cases
> What happens if you try to deref the same pointer twice
> DMP only activates on first deref
> Madge, it should activate on both
> It's history filter's fault
> History filter is turned off when more pointers are derefed in between
> Or after some CPU cycles have passed
> Do-not-scan means incomplete cache flush doesn't activate DMP
> Say you deref, flush the cache, then deref again
> If you only flush L1 and the history filter
> The second deref won't activate DMP
> You need to also flush L2
```

## Revealing whether an address is/isn't in cache

```
> Two techniques
> Flush + Reload is first
> If you can share the same process as the victim program
> Highly unlikely btw
> You can first flush its cache line
> Flushing can be done with cache thrashing
> Cache thrashing is creating a large array
> It must be bigger than the L1 or L2 cache the line is in, 8x
> Don't intentionally cache thrash
> It hurts the performance of programs
> Cache lines shared by different cores may contain shared variables
> Invalidating one of the shared variables will invalidate the cache line
> Cache lines contain other variables
> So it will invalidate the other legit variables
> Anyways, after the cache thrash, you let the victim program run
> You reload the cache to see if the victim stored any pointers in cache
> Does the cycles required to deref this match cache timings?
> Sometimes, there's noise
> That's why you need multiple iterations
> To ensure the timing distribution matches that of the cache
> Or that of DRAM's
> Second is Prime + Probe
> Doesn't require attacker to be in the same process
> Uses eviction sets
> Evictions sets are groups of virtual addresses
> The addresses map to the same cache set
> And remember, cache sets are indexed by page frames
> To find the eviction sets, you just iterate through page frames
> Prime + Probe tries to figure out if the eviction set was accessed
```

## RSA Attack

```
> Be RSA
> Implement CRT-based RSA
> Oops, compute c % p and c % q
> c is controllable by attacker
> Secret is the factorization of n = p * q
> If c looks like a pointer
> And c < p
> c % p results in DMP activation
> Can figure out the MSB of p by trying out different c's
> Binary search ftw
> Once get enough p bits, then launch Coppersmith's Attack
> n recovered
```

## Diffie-Hellman Attack

```
> Be Diffie-Hellman
> Oops, compute large exponentiation with windows
> Basically, when computing a^b % c
> Which the victim needs to do on attacker provided a
> If b is large, divide it into windows no larger than 8 bits long
> Called Montgomery Multiplication
> Do a^b0 % c
> Then do (a^b0)^2^8 % c
> Then multiply that by a^b1 % c, etc.
> Where b0, b1, ... are the windows
> The windows can be reversed
> If any of a^b0 % c is a pointer
> Or a^b0^(2^8) * a^b1 % c, ... is a pointer
> Then DMP is activated
> The secret window can be figured out
> And then the resulting b
> Just use Tonelli Shanks whenever exponent is even
> Duh, that basically solves stuff like x^2 = r mod q
> That's when x is unknown and r and q are known
> When exponent is odd, just do inverse mod
> Sometimes, there's no easy solution to Tonelli Shanks
> Then just regen the attacker input
```

## Kyber Attack

```
> Be Kyber
> Post-Quantum, but still vulnerable
> Secret is s
> Public key is (As + e, A)
> A is a matrix of polynomials in a polynomial ring
> e is an error polynomial
> Coeffs of polynomials are sampled randomly from Zmod q
> No coefficient is >= q
> No exponent is >= n
> This is what a polynomial ring does
> The polynomial ring Rq = Zq[x] / x^n + 1
> Every x^n + 1 turns into 0
> Kyber encryption takes pubkey, a 256 bit message M
> Outputs (u, v)
> u = A^Tr + e1
> v = t^Tr + e2 + mp
> t = As + e
> r is a seed for randomness
> e1, e2 are random polynomials
> mp is M converted to polynomial
> conversion looks like sum(Mi * floor(q/2) * x^i)
> Kyber decryption takes (u, v)
> Implicitly it has the secret (s, e)
> Should output m
> It calculates v - s^Tu
> = mp + e^Tr + e2 - s^Te1
> If v - s^Tu has coeff closer to floor(q/2) than to 0
> Its calculated Mi is 1
> Otherwise its Mi is 0
> Don't get how this works exactly
> Don't care, yet
> There is a decryption failure if coeff of e^Tr + e2 - s^Te1 >= floor(q/4)
> We'll know exactly because its output won't match our M
> So in theory
> We control e1, e2, M
> e1 is used to concentrate on a coeff of s
> If you want to find the 3rd coefficient, use e1 = (0, 0, 1, ...)
> e2 is used for activating DMP
> When e2 + s[0][0] >= floor(q/4), we can figure out s[0][0]
> Not all s can be recovered
> Due to the Top Byte Ignore
> The first 8 MSB bits of a pointer are ignored by DMP
> But partial s recovery is enough for lattice attacks to finish recovering s
```

## Dilithium Attack

```
> Be Dilithium
> Post-Quantum, but still vulnerable
> Uses the same LWE backbone as Kyber
> But instead, uses pubkey (As1 + s2, A)
> s1, s2 are both secrets
> Given (s1, M), it generates a signature (z, c)
> z = y + cs1
> c is a sparse polynomial depending on M
> sparse means few terms
> y is a random vector
> Issue is if z looks like a pointer
> Then y also looks like a pointer
> Then we can figure out s1
```

## Spectre

```
> This is like the Apple hardware-specific version of Spectre
> Spectre/Meltdown is a class of similar side channel attacks
> You can read any memory from other processes
> Basically, there's speculative execution
> When there's a branch
> You execute the next part of the branch that's most likely to execute
> That's called a commit
> This is especially great if reading memory is the branch
> Because reading memory is slow
> The commit will execute before the branch condition is evaluated
> You can trick the computer to always execute branches a certain way
> Then put a nuanced branch that actually hits the other way
> The computer will execute the wrong fork of the branch
> When it realizes it does this
> It clears the registers and rolls back the instruction pointer
> But it doesn't clear the cache
> Also, the execution of the commit doesn't check for Out Of Bounds reads/writes
> The buffer overflow protections don't get inserted until later
> Because speculative execution is supposed to be fast
> But you can't actually read the cache even if you write some valuable info in it
> So you basically do x[y[6]] in the malicious commit
> Where x is your legit array
> y is the array you're overrunning
> and 6 is the overflow which you use to read other process memory
> And now you can use a timing attack on x
> By checking derefs and seeing which index was fastest
```

## Resources

<a id="1">[1]</a>
https://gofetch.fail/

<a id="2">[2]</a>
https://www.howtogeek.com/891526/l1-vs-l2-vs-l3-cache/

L1, L2, L3 basics

<a id="3">[3]</a>
https://www.techtarget.com/searchstorage/definition/cache-memory#:~:text=DRAM%20is%20usually%20about%20half,to%20improve%20I%2FO%20performance.

DRAM basics

<a id="4">[4]</a>
https://medium.com/@ali.gelenler/cache-trashing-and-false-sharing-ce044d131fc0#:~:text=Cache%20Thrashing%20is%20a%20flaw,data%20in%20an%20undesirable%20way.

Cache Thrashing and False Sharing

<a id="5">[5]</a>
https://www.youtube.com/watch?v=YRohz9VO1YY

Spectre Attack