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

## Implementation attacks on block ciphers

### Side channel attacks

Measuring noise, time, power consumption for encryption and decryption.

### Fault attacks

Computing errors in the last round expose the secret key k. 

### Conclusion on implementation attacks

Don’t even implement these primitives yourself!

## Attacks on block ciphers

### Linear and differential attacks (Linear cryptanalysis)

Given many inp/out pairs, can recover key in less than exhaustive search (2^56 for DES)

Pr[m[i_1] XOR … XOR m[i_r] XOR c[j_j] XOR … XOR c[j_v] = k[l_1] XOR … XOR k[l_u] ] = 1/2 + epsilon

For DES, epsilon = 1/(2^(21)) because the fifth S-Box is too close to a linear function. 

How can we attack it to find key bits? 

Given 1/epsilon^2 random (m, c=DES(k, m) ) pairs then 
k[l_1, … , l_u] = MAJ [ m[i_1 , … , i_r] XOR c[j_j, …,j_v] ] with probability 97.7%. 

For DES, with 2^42 inp/out pairs, you can find k[l_i, …, l_u] in time 2^42. Roughly speaking: you can find 14 = 2 +12(from the 5th S-box) key bits this way in time 2^42.

There are 42 remaining bits in the key. Overall, the total attack time = 2^43 way better than 2^56! Better than exhaustive search.

Lesson: A tiny bit of linearity in S_5 lead to a 2^42 time attack! NEVER DESIGN YOUR OWN BLOCK CIPHER. 

### Quantum attacks

If you could build a quantum computer, a generic search problem that would be solved in O( |X| ), can be solved in O( |X|^(1/2) ) whatever the function is. 

Examples:
- DES = 2^28
- AES-128 = 2^64
- AES-256 = 2^128 

## AES

### History 

- 1997: NIST publishes request for proposal 
- 1998: 15 submissions (5 claimed attacks)
- 1999: NIST chooses 5 finalists
- 2000: NIST chooses Rijndael as AES (designed in Belgium)

Key sizes = 128, 192, 256 bits. Larger keys: slower but thought to be more secure.

Block size = 128 bits

### Design

AES is a substitution-permutation network. In a Feistal network, half of the bits are not changed in every round. In a subs-perm network, all bits are changed on every round. 

AES-128 schematic 

![AES-128](http://cl.ly/Tc1N/Screen%20Shot%202014-01-28%20at%2014.36.28.png)

AES operates on 128 bits, a 4x4 matrix, each cell containing a byte. Then we XOR with the first round key, apply the round function, x10 and then we get the output. The keys are coming from the 16 bytes AES key using key expansion.
 
![AES-128-RoundFunctions](http://cl.ly/TbLk/Screen%20Shot%202014-01-28%20at%2014.36.35.png)

Overview of the round function: 
- Byte substitution: one byte S-Box (256 byte table). We take the current cell as an index into the lookup table, and the value is the output.
- Shift row step: We shift the second row from 1 position, third row by 2 positions and last row by 3 positions.
- Mix column: We apply a linear transformation to each of the communes independently. 

### How to use AES

If you want to send an implementation over a network. Don’t send precomputed table but algorithm to compute it. And then compute them upon receival. 

AES is implemented in hardware. aesenc, aesenclast: one round of aes. aeskeygenassist, perform key expansion. 14 times faster than software. 

### Attacks on AES

Best key recovery attack: four times better than exhaustive search. 128key => 126 key.

Related key attack on AES-256: If related keys => 2^99 security! *Importance to choose keys at random*.

## Building block ciphers from PRG

Can we build a PRF from a PRG? 

Let’s start with a PRG G such that G:K -> K^2 be a secure PRG. 

Define 1-bit PRF F:Kx{0,1} -> K as F(k, x in {0,1}) = G(k)[x]

If G is a secure PRG, then F is a secure PRF on {0,1}^n => Not used in expanded mode due to performance reasons. 

Thanks to the Luby-Rackoff theorem, we know that we can thus make a secure PRP with a 3-round Feistal network.

## Notes and review

A block cipher maps n bits of input to n bits of output. 

PRF -> function, X -> Y, doesn’t have to be the same. 
PRP -> One to one revertible function. Domain X=Y. Key concept to build a block cipher.

Any secure PRP is also a secure PRF if |X| is sufficiently large.

Lemma: Let E be a PRP over (K,X) then for any q-query adversary A: |Adv_{PRF} [A,E] - Adv_{PRF} [A,E] | < q^2 / 2|X|
When X is large, the ratio will be negligible.

From now on, we consider AES or 3DES as secure PRPs.

## Security for one-time key

Let’s start with a threat model (one-time keys) defined as follows:
- Adversary’s power: Adv sees only one cipher text
- Adversary’s goal: Learn info about PT from CT (semantic security)

Reminder: semantic security for a one-time key. The attacker, if given c_0 and c_1 and m_0 and m_1 can’t know which one is the result of what message. 
Adv_{SS} [A, OTP] = | Pr[ EXP(0)=1 ] - Pr[ EXP(1) = 1 ] |

## Security for many-time key

Why? Many applications: Filesystems or IPSec, encrypts a lot of traffic with the same key. 

When we use a key more than once, the adversary sees many cipher texts with the same key. 

Adversary power: chosen-plaintext attack, the attacker can obtain the encryption of arbitrary messages of his choice (How does it work IRL? You can email someone, email will be stored encrypted on disk and boom you have m and c).

Adversary goal: Break semantic security

Semantic-security for many-time key is defined exactly as semantic security for a one-time key but he can repeat any of the messages in the challenge that the attacker can choose. ==> Chosen Plaintext attack.

All the deterministic encryption schemes we’ve seen before are broken under CPA. 

So how do we fix this?

1) Randomized encryption: encrypting same message twice gives different cipher text. Ciphertext must be longer than plaintext. size(CT) = size (PT) + “#random bits”

2) Nonce-based encryption. We define a **nonce** as a value that changes from message to message. The pair (k, n) should NEVER be used more than once. The nonce can conveniently be a counter (if in-order and reliable transmission channel, no need to transmit nonce). If same key used by multiple machines, the nonce space needs to be very big and picked at random (easier to implement a “stateless” protocol)

## Modes of operation

Goal: Build a secure encryption from a secure PRP

### ECB (Electronic Code Book) - One time key

ECB is badly broken. It works by breaking down the message into n blocks of size of the block cipher and then encrypt each of the parts individually. Issue, the attacker learns when a two segments have the same value. (if m_1 = m_2 -> c_1 = c_2)

ECB is not semantically secure.
![ECB’s advantage is 1!](http://cl.ly/Tdm9/Screen%20Shot%202014-01-30%20at%2011.28.png)

### Deterministic counter mode from a PRF F (eg. AES) - One time key

E_{DETCTR} (k,m) = We build a stream cipher from a PRF.

![Deterministic Counter Mode](http://cl.ly/Te5t/Screen%20Shot%202014-01-30%20at%2011.37.39.png)

### CBC (Cipher Block Chaining with a random IV) - Many time key (CPA security)

#### IV-based encryption

When we start encrypting the first block, we pick a random IV (initialization vector of length one bloc).  We XOR the first message with it before encrypting. 
IV is publicly known and prepended to the cipher text. 
Chaining is done by for the next block XORing the cipher text of the first block with the new message and then encrypting. 

What security does it provide? 

It does provide semantic security:

Adv_CPA [A , E_{CBC} ] =< 2 * Adv_PRP [B, E] + (2 q^2 L^2 / |X| = error term, needs to be negligible) 

**CBC is secure as long as q^2L^2 << |X| where L is the length of the messages and q is the number of times we used q to encrypt messages.**

Applied to AES, this means that after 2^48 blocks, we need to replace the key. 

Cipher is not CPA secure if the IV is predictable. 

![IV-based encryption](http://cl.ly/ThMj/Screen%20Shot%202014-02-01%20at%2021.47.48.png)

#### Nonce-based encryption

Cipher-block chaining with unique nonce. (no need to include in first cipher text)

If nonce is not random, it needs to be xored with first block.

![Nonce-based CBC](http://cl.ly/Tgje/Screen%20Shot%202014-02-01%20at%2021.47.34.png)

#### Padding 

In TLS, you pad the n remaining bytes with the number n. If no pad is needed, add a dummy block. 
 
### Randomised Counter-mode (superior to CBC) 

Unlike CBC, randomised counter-mode doesn’t need a secure block cipher (PRP) but works with a secure PRF because we’re never going to invert the function F. 

How does it work?

We pick a random IV, then we XOR the messages blocks with F(k,IV + message block number)

Nonce based counter mode: IV = [ 64-bit nonce | 64-bit counter (starts at 0 for every message)]

Note that we can encrypt a maximum of 2^64 blocks per nonce because otherwise the counter overflows and the pad would be used a second time.

We can use counter-mode for more blocks than CBC because  the adversary’s advantage is 2q^2L / |X| < CBC’s advantage. 

For AES, we can encrypt 2^64 AES blocks  with the same key with semantic secrecy. 
 
Advantage: It’s parallelizable! Fast encryption! And is so much better than CBC.

![CBC vs Counter](http://cl.ly/TgXt/Screen%20Shot%202014-02-01%20at%2022.38.13.png)

# Message Integrity 

## Message Authentication Codes (MAC)

A MAC is defined as (S,V) and defined over (K, M, T) is a pair of algorithms:
- S(k,m), the signing algorithm, outputs t (tag) in T 
- V(k,m,t), the verification algorithm, outputs yes or no

— 

Consistency requirement: for every message and for every key: V(k, m, S(k,m)) = yes

—

Integrity requires a shared key between Alice and Bob. Algorithms like CRC do detect random errors but not malicious errors! 

### Secure MACs 

The attacker’s power : chose message attack:
- for m_1 … m_q attacker is given t_i <- S(k,m_i)

The attacker’s goal: Existential forgery
- produce some new valid message/tag pair (m,t): (m,t) different than any pair that is given to him.

What does this mean?
- The attacker cannot produce a valid tag for a new message.
- Given (m,t) attacker cannot even produce (m,t’) for t != t’

—

Definition: I=(S,V) is a secure MAC if for all “efficient” A: Adv_{MAC} [A, I] = Pr[Chal. outputs 1] is negligible.

—

A secure PRF => Secure MAC if the output space of the PRF is big enough. In practice, 80-bits is good security for a PRF. 

We can use AES for 16-byte messages but how can we convert a MAC for small inputs and scale them for bigger inputs. The output of a n bit PRF can be truncated. 

### (encrypted) CBC-MAC

Let F:K x X -> X be a PRP, define a new PRF F_ECBC:K^2 x X^{=< L} -> X

![CBC-MAC construction](http://cl.ly/The8/Screen%20Shot%202014-02-03%20at%2015.36.37.png)

### NMAC

Let F:K x X -> K be a PRF, define a new PRF F_NMAC : K^2 x X^{=<L} -> K.

![NMAC Construction](http://cl.ly/TiX2/Screen%20Shot%202014-02-03%20at%2015.44.55.png) 

![CBC-MAC vs NMAC](http://cl.ly/Tiod/Screen%20Shot%202014-02-03%20at%2015.36.37.png)

### MAC padding

Errors in padding can have disastrous consequences, just imagine if someone can forge a banking transaction with an additional 0! 

So, how do we pad?  Padding must be reversible (1 to 1) to make sure it’s unique. We pad with “100…0”. Add a new dummy block if the message size is already a multiple of the block size.

### CMAC

Variant of CBC-MAC where no additional encryption step is necessary and no need to add a dummy block. CMAC uses key = (k, k_1, k_2) where k_1 and k_2 are derived from K. 

![CMAC construction](http://cl.ly/TiW6/Screen%20Shot%202014-02-03%20at%2017.07.21.png)

### PMAC - Parallel MAC

All the PRFs seen so far for MACs are sequential. PMAC is parallel and incremental (if one block changes, no need to recompute everything). 

![PMAC Construction](http://cl.ly/Thwf/Screen%20Shot%202014-02-03%20at%2017.23.06.png)

### One-time MAC

![One Time MAC example](http://cl.ly/Til8/Screen%20Shot%202014-02-03%20at%2017.51.37.png)

### Many-time MAC

![Many Time MAC](http://cl.ly/TiMx/Screen%20Shot%202014-02-03%20at%2017.53.44.png)

## Collision Resistance

Let H:M -> T be a hash function (|M| >> |T|)

A **collision** for H is a pair m_0, m_1 in M such that: H(m_0) = H(m_1) and m0 != m1

A function H is **collision resistant** if for all efficient algorithms A: Adv_CR [A,H] = PR[A outputs collision for H] is negligible. 

If we have a collision-resistant function, we can build MACs for bigger messages from secure MACs that work on smaller messages.

### Protecting file integrity

If we have a public read-only space and no key, we can use a collision resistant hash function to verify integrity of a package.

### Birthday attack

Using the birthday paradox, we can predict collisions with probability: when n = 1.2 * B^{1/2} then Pr[collision] => 1/2
with n, number of items in attack set and B, number of items in universe.

### Using Hash-functions

Use SHA-512 is recommended. 

### The Merkle-Damgard Paradigm

![Merkle Damgard](http://cl.ly/Tiap/Screen%20Shot%202014-02-04%20at%2001.27.14.png)

Security guaranteed by theorem that claims that if h is collision resistant then so is H

### Compression functions

#### Davies-Meyer compression function based on block ciphers

The Davies-Meyer compression function: h(H,m) = E(m, H) XOR H

Theorem: Suppose E is an ideal block cipher. Finding collisions h(H,m)=h(H’,m’) takes O(2^(n/2)) - same as birthday attack, best possible, evaluations of (E,D).

### SHA-256

SHA-256 is a Merkle-Damgard function that uses the Davies-Meyer compression function and where the block cipher used is SHACAL-2 (512-bit keys) and block size of 256 bits. 

### HMAC - MAC from SHA256

HMAC: S(k,m) = H( k XOR opad, H( k XOR ipad || m))

![HMAC In Pictures](http://cl.ly/Ti2P/Screen%20Shot%202014-02-04%20at%2001.47.50.png)

ipad and opad are 512 bits constants. 

### Verification timing attacks

Make sure that verification is constant time.

1) Iterate through every set of bytes. Be careful of compiler optimisations!
2) Compute correct MAC, we hash the correctly computed MAC again and then compare byte by byte. This way the attacker doesn’t know what is being compared!

Lesson: Don’t implement your own crypto!

