# Crypto notes - Stanford

From the introduction, really excited about: homographic encryption (computations on cipher text!)+ Zero-Knowledge proof (proving someone that I know p and q without revealing anything else than N)  

# History

Symmetric ciphers:
- substitution ciphers (breaking by entropy attack)
- Caesar cipher (bad substitution cipher because no key (if you know the number of shifts, you win))
- Vigener cipher (have a key that's concatenated as long as there is plaintext, then take CipherText = (key + plaintext mod 26)). When we discover the key size, we can thus perform an entropy attack on every letter.
- Rotor Motor :
	- Hebern: Rotor-powered substitution table -> statistical attacks
	- Enigma (2^18 keyspace) - broken by cipher text attacks 

- DES (1974, NIST standard)
	- 2^56 keyspace -> can be bruteforced these days, not to be used anymore
	- blocks size 64 bits

Now: AES, SALSA20

# Discrete Probability 

Not much new material covered => See chapter in DS course for more info.

Random variable, events, independence, XOR ...

XOR has the amazing property of always adding entropy, never removing it. 

## Birthday paradox 

when n = 1.2 x U^(1/2) ==> Pr[there exists two similar elements] >= 1/2

# Stream ciphers

## Information theoretic security 

A **cipher** is defined over a triple ( the key space, message space, cipher space) and does provide two functions E and D in such a way that D(k, E(k,m)) = m.

E is sometimes randomised but D is always deterministic. 

**The Vernam Cipher** is a cipher where the key is as long as the message where the cipher text is obtained by E(m,k)=(message XOR key) = cipher text and D(c,k) = c XOR k = message.

Vernam is not useful in practise because if we have a secure channel to send the keys over, we could use that to send the message, so what’s the point of the cipher, right?

A cipher (E,D) over (K, M, C) has **perfect secrecy** if for every m_0 and m_1 in M, length of m_0 is equal to the length of m_1 and for every c in C, Pr[E(k,m_0) = c] = Pr[E(k, m_1)=c].
Perfect secrecy means that there are no cipher text only attack!

If one cipher has perfect secrecy => |K| >= |M|, the key_length >= message_length.

## Stream ciphers

Idea: replace random key by pseudorandom key. 

For the stream ciphers, we introduce a PRG: Pseudo-random generator.

A **PRG** is a deterministic function G:{0,1}^s (seed space) -> {0,1}^n, n>>s.

In stream ciphers, seed = key and define the following operations.
E(k,m) := m XOR G(k)
D(k,c) :=  c XOR G(k)

## PRG
A PRG must be unpredictable for a stream cipher to be secure. 

We say G:K -> {0,1}^n is predictable if there exists an efficient algorithm A and there exists an i: 1<i<n-1 such as Pr[A(G(k))|i,…,i = G(k)|i+1] >= 1/2 + epsilon. (for some non-negligible epsilon).

In short, a PRG is unpredictable: for all i, there is no efficient algorithm A, that can predict the i+1 bit following the prefix.

NEVER USE Weak PRGs: Linear congruential generators (glibc random). 

Negligible and non-negligible - In practise, epsilon is a scalar such as the event is not likely to happen. In theory, non-neg is defined as there exists d: epsilon(polynomial lambda) >= 1/(lambda^d) infinitely often. 

TLDR’: The function is negligible if it’s less than all polynomial fractions. 

## Attacks on OTP / stream ciphers

The two time pad is insecure. 

Example: 
c_1 = m_1  XOR PRG(k)
c_2 = m_2 XOR PRG(k)

If an attacker gets c_1 and c_2, he can perform c_1 XOR c_2 = m_1 XOR m_2

The english language (and ASCII encoding) has enough redundancy to find m_1 and m_2. 

Vulnerabilities found in Project Venona (1941-1946), MS-PPTP (Windows NT) and WEB where the same pad would be used every 16M frames. Even worse, most Wi-Fi cards do reset their IV back to zero on restart, causing the two time pad to appear way quicker because IC is concatenated with a long-term identity key.

In disk encryption, issue of the one time pad is **malleability**. Because OTP *does not provide integrity-checking*, cipher text could be altered without the user knowing. If you know the difference between two strings, you can XOR that difference on the cipher text without knowing the key and it be changed in the cipher text without you having to know the key.

## Modern broken stream ciphers
 **RC4** (1987) - stream cipher designed to be implemented in *software*- takes a variable size seed, expands it to a broader number of bits and then generates 1 byte (of pseudorandom) per round.  Weaknesses: 1) ex. Pr[2nd byte = 0] = 2/256 if seed = 128bits and expansion to 2048bits. (256th first bytes are biased) 2) Pr[(0,0)] is getting bigger than it should after a few gigs of data. Should not be used anymore.

**CSS** is a stream-cipher used for encrypting DVDs. It’s badly broken. It was designed to be implemented on *hardware* and based on a mechanism called **LFSR** (linear feedback shift register), also used for GSM encryption (A5/1,2) and Bluetooth (E0). LFSR-derived stream ciphers are all badly broken but hard to fix because implemented in hardware.
 
LFSR is works: 
- We have a register of n bits.
- At every clock cycle, we shift the entries of the registry to the left, the last bit falls off and the first bit becomes the result of the XOR of all bits of the register of the previous state.
- Seed = initial state of the register.

CSS: 
- seed = 5 bytes = 40 bits (because crypto-regulations in the US only allowed exports of crypto using 40 bits keys!)
- 2 LFSRs (17-bit and 25-bits). The first one is seeded with (1 + first 2 bytes of the key) and the second one is seeded with (1 + last 3 bytes of the key). Each of the LFSRs do produce 8 bits outputs (in 8 cycles) that are then added and mod 256 + carry of previous block. And this outputs one byte per round that is then XORed with the byte of the movie we are trying to encrypt.

CSS can be broken in 2^17.

## Better modern stream cipher

Better PRG are coming from the **eStream** project (2008) .
PRG: {0,1}^s x R (nonce) -> {0,1}^n

A **nonce** is a non-repeating value for a given key.

E(k,m,r) = m xor PRG(k,r)

The pair (k,r) is never used more than once => You can reuse the key because a nonce make the key unique. 

## Let’s do some Salsa!

**Salsa20** was designed to be easy to implement on both software and hardware. It was part of the eStream project and was submitted by DJB.

Salsa20: {0,1}^{128 or 256} x {0,1}^64 -> {0,1}^n (max n=2^73)

Salsa20(k,r) := H(k, (r,0) ) || H(k, (r,1) ) || … 

How is that function H(k, (r,i) ) defined?

For the 128 version of Salsa20, we start with 64 bytes defined as in this slide. T_0…3 is defined in the Salsa20 specification. You perform 10 rounds of an invertible function h and do add all of the them word by word and you get a 64 byte output. 

![Salsa20](http://f.cl.ly/items/0841112C3n1v0Y3m1o2t/Screen%20Shot%202014-01-21%20at%2022.26.38.png)

There are no significant attacks known on Salsa20. Very fast stream cipher in both software and hardware. In crypto++, RC4 = 126 MB/s and Salsa20=643 MB/s. 

## PRG Security

The seed space is really small compared to the universe {0,1}^n. A pseudorandom generator outputs would look indistinguishable from a uniform distribution on {0,1}^n. 

One way to test if a PRG is pseudorandom we do perform statistical tests. How do we evaluate if a statistical test is good? We define the advantage of a statistical test A over the generator G: Adv[A,G] = | Pr[A(G(k))=1 ]- Pr[A(r) =1] | (between 0,1) 

If the advantage happen to be close to 1 it means that the generator behaved differently from the random distribution. This statistical test can distinguish the difference from random. If the advantage is close to 0, A can not distinguish the generator from random. 

The test A is said to break the generator if it has an advantage close to 1. 

If the statistical test always outputs 0, it’s advantage is 0. 

We define a **secure PRG** such as, for all statistical tests A, Adv[A,G] is negligible.

So, can we prove it? No we can’t. ( p != np) but we have heuristic candidates.
—
If PRG is predictable => PRG is insecure (can be proved easily by using the advantage definition of a secure PRG). If next-bit predictors cannot distinguish G from random then no statistical test can!

## What is a secure cipher?

We can adapt the definition of perfect secrecy from Shannon with the concept of computational indistinguishability. 

For every message m_0, m1 in M: E(k, m_0) is computationally indistinguishable from E(k, m_1). 

A cipher is **semantically secure** if for all efficient adversary A, Adv[A, cipher] is negligible. 

# Block Ciphers

A block cipher maps n bits of inputs to n bits of output. 
Examples : 
- 3DES: n=64bits, k=168 bits
- AES: n=128bits, k = 128,192,256 bits

Block ciphers are typically built by iteration with a round function. It takes as input the  round key and the message. 

Block ciphers are considerably slower than stream ciphers. (+/- 6x slower if you compare Salsa to AES)

## Pseudo Random Function (PRF)

A PRF is defined over (K, X, Y) with F: K x X -> Y such  that there exists an efficient algorithm to evaluate F(k,x). 

Note: A PRF doesn’t need to be revertible. 

## Pseudo Random Permutation (PRP)

A PRP (block cipher) is defined over (K,X) with E: K x X -> X such that:
- Exists efficient and deterministic algorithm to evaluate E(k,x)
- The function E(k, ・) is one-to-one.
- Exists an efficient inversion algorithm D(k,y)

Examples:
- 3DES: K x X -> X where X = {0,1}^64 , K = {0,1}^168
- AES: K x X -> X where K = X = {0,1}^128

Any block cipher or PRP is also a PRF : A PRP is a PRF where X = Y and is efficiently invertible.

### Secure PRF

Let F: K x X -> Y be a PRF
- Funs[X,Y]: the set of all functions from X to Y
- S_{F} = { F(k,・) s.t. k ∈ K} ⊆ Funs[X,Y]

Intuition: A PRF is secure if a random function in Fun[X,Y] is indistinguishable from a random function in S_F.

### PRF gives us a PRG

Let F: K x {0,1}^n -> {0,1}^n be a secure PRF. Then the following G: K -> {0,1}^{nt} is a secure PRG: G(k) = F(k,0) || F(k,1) || … || F(k,t)
Key property: parallelizable
Security from PRF property: F(k,・) indistinguishable from a random function f(・)

## DES (Data Encryption Standard)

Early 1970: Horst Feistel designs Lucifer at IBM. key-len = 128 bits, block-len = 128 bits.
1973: NBS (old name of NIST) asks for block cipher proposals. IBM submits variant of Lucifer. 
1976: NBS adopts DES as a federal standard. key-len = 56 bits. block-len = 64 bits.
Note: This is yet another example where standard bureaus do weaken cryptography. 
1997: DES broken by exhaustive search
2000: NIST adopts Rijndael as AES (Advanced Encryption Standard) to replace DES.

Widely deployed in banking (ACH) and commerce.

### Feistel Network
Given functions f_1, …, f_d: {0,1}^n -> {0,1}^n 
Goal: build invertible function F:{0,1}^{2n} -> {0,1}^{2n}
A Feistel Network mapping a 2n bit input to 2n bit output:
Encryption
L_i = R_{i-1}
R_i =  L_i XOR f_i (R_{i-1})
Decryption
L_i = f_{i+1} (L_{i+1}) XOR R_{i+1}
R_i = L_{i+1}

![Construct Inverse](http://cl.ly/TZkE/Screen%20Shot%202014-01-26%20at%2013.42.32.png)

Feistal networks are a general method for building invertible functions (block ciphers) from arbitrary functions. And it’s used in many block ciphers but not AES. 

The Luby-Rackoff Theorem proves that if I take a secure PRF and let it go through 3 rounds of a Feistal network, the result is a secure PRP. Formally
f: K x {0,1}^n -> {0,1}^n a secure PRF => 3-round Feistel F: K^3 x {0,1}^{2n} -> {0,1}^{2n} is a secure PRP. 

### DES is a 16 round Feistel network
f_1, … , f_1: {0,1}^32 -> {0,1}^32, f_i (x) = F{k_i, x} where k_i is a round key from the key expansion. For decryption, the algorithm is the same but you use the round keys in reverse-order. 
F(k_i, x) is taking a 32-bit value x and a 48-bit round key k_i.

![F function](http://cl.ly/TYeV/Screen%20Shot%202014-01-26%20at%2014.03.08.png)

### S-boxes

S-boxes are just lookup tables. If S-Boxes were linear (if the S-boxes could be written as a Matrix vector product), the entire cipher would be linear and quickly broken. If S-Boxes would be random, it would result in an insecure block cipher (key recovery after 2^24 outputs). 

So what are the creators of DES advising to make S boxes lookup tables? 
- No output bit should be close to a linear function of the input bits
- S-boxes are 4 to 1 maps.
- …

### Exhaustive search on DES

Goal: given a few input/output pairs (m_i, c_i = E(k, m_i)), find key k. 

Lemma: Suppose DES is an **ideal cipher** (2^56 random invertible functions). Then for all m,c there is at most one key k such that c = DES(k,m).

With two input-output pairs, the probability that the key is unique is very close to one for both DES and AES. Hence, two input/output pairs are enough for exhaustive key search. 

RSA issued a challenge to break DES exhaustively:
- 1997: Internet Distributed Search = 3 months
- 1998: EFF machine (deep crack) = 3 days
- 1999: combined search = 22 hours
- 2006: COPACOBANA (FPGA) = 7 Days (cheap!)

**DES is broken**

How do we make DES more expensive to do exhaustive search? More rounds!

#### Method 1: Triple DES. 

Triple DES is also 3x slower for encryption :( 

The key-size of 3DES is 3*56 bits = 168 bits but does not provide 168 bits security only 118 bits. -> Meet in the middle attack

For 2DES: exhaustive search is formulated as finding (k_1,k_2) such that E(k_1, E(k_2, M)) = C. A meet-in-the-middle attack is E(k_2, m) = D(k_1,c)

Step 1: Build table of pairs (k0…kN ; E(k0, M)…E(KN, M)
Step 2: For all k ∈ {0,1}^56: test if D(k,C) is in 2nd column. 
Step 3: If found, then E(k^i, M) = D(k,c) => (k^i, k) = (k_2, k1) 

Running time = Time = 2* 2^56  * log 2^56 < 2^63 << 2^112 

Same attack on 3DES: Time = 2^(118), space = 2^56

#### Method 2: DESX

E: K x {0,1}^n —> {0,1}^n a block cipher. EX((k,1,k2,k3), m) = k_1 XOR E(k_2, m XOR k_3)
key-length = 184 bits. Attack known in 2^120.

