# Crypto Challenge 6: Rivest Shamir Adleman
## Investigation
So this challenge provided a zip file that contained 10 files. I began by looking at the two python files "encrypt.py" and "generate_keys.py". The encrypt file did not tell me anything new because the challenge description informs us that this is someone's implementation of RSA. So I move onto the generate_keys file. While stepping through the file I noticed vulnerablity in the key generation.
`

    primes = []
    for i in range(2):
        print(f"Searching for prime {i+1}")
        while True:
            print("Generating new prime - ", end="")
            p = prime_candidate(1024)
            print(str(p)[:10] + "... - ", end="")
            if new_primality_test(p):
                break
            print("not prime")
        print("prime!")
        primes.append(p)

    primes[0] = int(open("component.txt").read())
    
`
The last line is where the problem lies. For the RSA cryptosystem to work correctly you need two unique large prime numbers. But here you see that one of those prime numbers is being read from one of the included files. Meaning that one prime will not be unique. Thus the whole system can be broken. After looking through the two json files we have the public key which is comprised of the values n and e. Then remember that problematic prime number? Opening the file component.txt gives us the prime number we will call p. With these three values n, e, and p we can crack the encoded text that is contained in the file "secrets.txt.enc".
## Mathematics
### ciphertext = (plaintext ^ e) % n
### plaintext = (ciphertext ^ d) % n
Above is how encryption and decryption occurs in RSA respectively. The public key which everyone can see is the two values n and e. What you know as the private key is that value d. Knowing d breaks the system. And in this situation we can calculate d. The following will be a walkthrough of that calculation.
### d = modular_inverse( e, phi_n )
This is the function that will give us d. You don't need to know what exactly modular inverse does to break RSA. All you need is e and a new value phi_n to plug into any modular inverse function. So as has been established we know e but what is this new phi_n value?
### phi_n = lcm( p-1, q-1 )
The value of phi_n is the lowest common multiplier between a value p-1 and q-1. As stated earlier we know p but not q. Once we know q the rest is just as simple as plugging values in.
### n = p * q
This is where the strength of RSA lies if implemented correctly. When given n it is pretty much impossible to find p and q if they are unique and large as hell. But we know p. So finding q is as simple as q = n / p. Once q is calculated we go back to phi_n and find that. Then we use wine to run the provided yafu program to quickly calculate the modular inverse of e and phi_n. The result is d which can be used with the provided python decrypt file to output the decrypted text! 

