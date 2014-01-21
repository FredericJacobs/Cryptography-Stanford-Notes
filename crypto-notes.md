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

--- Birthday paradox 

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

 