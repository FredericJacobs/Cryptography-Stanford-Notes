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

 