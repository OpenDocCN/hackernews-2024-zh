<!--yml
category: 未分类
date: 2024-05-27 14:32:39
-->

# Implementing RSA in Python from Scratch

> 来源：[https://coderoasis.com/implementing-rsa-from-scratch-in-python/](https://coderoasis.com/implementing-rsa-from-scratch-in-python/)

Please note that it is essential for me to emphasize that the code and techniques presented here are intended solely for educational purposes and should never be employed in real-world applications without careful consideration and expert guidance.

At the same time, understanding the principles of RSA cryptography and exploring various implementations is valuable for educational purposes and understanding how to code encryption methods, building secure encryption systems for practical use requires adherence to industry standards, best practices, and thorough security assessments. An inadequate implementation or improper usage of cryptographic techniques can lead to severe vulnerabilities, jeopardizing the confidentiality and integrity of sensitive data.

* * *

I'm sure many programmers, particularly web developers have heard of the RSA cryptography system. RSA is an asymmetric cryptography system, meaning that one key is used for encryption and the other for decryption. I've seen a lot of articles explaining the general principles of asymmetric cryptography, but I have not seen any that give easy-to-understand explanations of the mathematical background behind these algorithms, so I decided to write this article.

## **The Math Behind RSA**

For start, as I've mentioned in the paragraph above, to transmit an RSA-encrypted message, you need 2 keys. One that can only encrypt and one that can decrypt. Let encryption key be denoted as `e`, the decryption key will be denoted as `d` and the message will be denoted as `m`. The key principle behind RSA is that in the following notion:

```
(m^e)^d ≡ m (mod n)
```

The difficulty of finding `d`, knowing `e` and `n` can be very hard if these numbers are chosen properly (as will be demonstrated below).

But first, we need to introduce some new concepts in number theory.

*   `a ≡ b (mod n)` means that there is an integer `x` such that `a + nx = b`. A more intuitive explanation is that the reminder of `a / n` equals the reminder of `b / n`.
*   `gcd(a, b)` is the greatest number that divides both `a` and `b`.
*   `lcm(a, b)` is the smallest number that is a multiple of both `a` and `b`.
*   `λ(n)` is a number such that `x^λ(n) ≡ 1 (mod n)` for any `x` such that `gcd(x, n) = 1`. From this you can conclude that `x^a ≡ x^b (mod n) if a ≡ b (mod λ(n))`

## Generating keys

Let's make `n = pq` where `p` and `q` are 2 prime numbers. Since `λ(p) = p - 1` and `λ(q) = q - 1` (lookup Fermat's little theorem proof for why), `λ(n) = (p - 1) * (q - 1)`.

Now we have to solve `ed ≡ 1 (mod λ(n))`. `e` can be some prime number (usually 65537) and d can be calculated using extended Euclidian's algorithm (will be written and explained in the code section) from the equation `ed + xλ(n) = gcd(e, λ(n))` (`d` and `x` are coefficients to be calculated).

`pair (e, n)` is used for encryption (`(m^e) % n`) and `pair (d, n)` is used for decryption (`(m^d) % n`)

After computing the keys, p and q can be thrown away. Notice that without `p` and `q`, finding `λ(n)` would mean factorizing `n`, which is not an easy problem to solve for values of n up to 2^2048, which are regularly used.

## **Implementing RSA in Python**

First to list procedures and their steps:

#### keys generation:

*   find 2 random prime numbers, `p` and `q`
*   compute `n = p * q` and `λ(n) = (p - 1) * (q - 1)`
*   make `e` equal some prime number, e.g. `e = 35537`
*   compute `d` from equation `ed + λ(n)x = gcd(e, λ(n)) = 1` using Extended Euclidian Algorithm (from this point on we will call it `eucalg`)

##### Encryption:

*   divide the message into sections (of 256 bits if n is 2048-bit)
*   each section (denoted as `m`) is encrypted as `(m^e) % n`

#### Decryption:

*   divide the message into sections, same as above
*   each section (denoted as `m`) is decrypted as `(m^d) % n`

## Extended Euclidian Algorithm

This algorithm relies on the fact that if `x` divides both `a` and `b`, there will be a pair of coefficients `(k, l)` such that:

`a * k + b * l = x`

The algorithm finds these coefficients (and `x`) by repeatedly subtracting the lesser argument from the bigger one, until the lesser one becomes 0\. Instead of repeatedly subtracting, you can calculate how many times `b` can be subtracted from `a` (`k = a // b`) and then subtract `b*k`. This optimization makes the algorithm run in O(log max(a, b)) which is a lot quicker.
Below is an implementation of the algorithm in Python:

```
def eucalg(a, b):
	# make a the bigger one and b the lesser one
	swapped = False
	if a < b:
		a, b = b, a
		swapped = True
	# ca and cb store current a and b in form of
	# coefficients with initial a and b
	# a' = ca[0] * a + ca[1] * b
	# b' = cb[0] * a + cb[1] * b
	ca = (1, 0)
	cb = (0, 1)
	while b != 0:
		# k denotes how many times number b
		# can be substracted from a
		k = a // b
		# a  <- b
		# b  <- a - b * k
		# ca <- cb
		# cb <- (ca[0] - k * cb[0], ca[1] - k * cb[1])
		a, b, ca, cb = b, a-b*k, cb, (ca[0]-k*cb[0], ca[1]-k*cb[1])
	if swapped:
		return (ca[1], ca[0])
	else:
		return ca 
```

Notice that the function above can produce negative numbers for coefficients, so in case that happens, all that we need to do is set `d` to `λ(n) - d`.

## Fast Exponentiation with Modulo

Some might suggest to just use `(b ** e) % n`, but this approach is not very good time and memory -wise because `b ** e` can be very big despite only the last 2000 bits or so being needed. The solution is to reimplement exponentiation with calculating modulo after every division.

Exponentiation implementation below has logarithmic time complexity. Instead of iterating from 1 to `e` and multiplying `r` (result) by `b`, it iterates from the most significant bit of `e` to the least significant bit of `e`, and at each bit it does `r = r*r + bit`. This works because if `r` equals `b^x` and you're appending a bit to the end of x, new x will be `x * 2 + bit`.

```
def modpow(b, e, n):
	# find length of e in bits
	tst = 1
	siz = 0
	while e >= tst:
		tst <<= 1
		siz += 1
	siz -= 1
	# calculate the result
	r = 1
	for i in range(siz, -1, -1):
		r = (r * r) % n
		if (e >> i) & 1: r = (r * b) % n
	return r 
```

## Generation/Encryption/Decryption

Keys generation, encryption and decryption have all been explained in the math section and the code below is simply implementation of that.

```
def keysgen(p, q):
	n = p * q
	lambda_n = (p - 1) * (q - 1)
	e = 35537
	d = eucalg(e, lambda_n)[0]
	if d < 0: d += lambda_n
        # both private and public key must have n stored with them
	return {'priv': (d, n), 'pub': (e, n)}

def numencrypt(m, pub):
	return modpow(m, pub[0], pub[1])

def numdecrypt(m, priv):
	return modpow(m, priv[0], priv[1]) 
```

## Code Testing

I stored the code into a file named rsa.py and run the following in Python shell:

```
>>> import rsa
>>> keys = rsa.keysgen(31337, 31357)
>>> keys
{'priv': (720926705, 982634309), 'pub': (35537, 982634309)}
>>> priv = keys['priv']
>>> pub = keys['pub']
>>> msg = 80087
>>> enc = rsa.numencrypt(msg, priv)
>>> enc
34604568
>>> dec = rsa.numdecrypt(enc, pub)
>>> dec
80087
>>> 
```

## **The Conclusion**

In the end of writing this article I realized that implementation of a usable Python RSA program is more complicated than I initially speculated. Because of that, I decided to split the topic up in multiple articles. With the code presented in this article you have all core parts of RSA written. But you still need a random prime generator and plaintext encryption (`numencrypt` and `numdecrypt` are suitable only for numbers `m` smaller in size than `n`). All these will be included in the next article that will be published shortly.

* * *

If you want to continue reading, we also have a Part 2 and even a Part 3 to this article for those who are interested in Implementing RSA in Python from Scratch.

 [Implementing RSA in Python from Scratch (Part 2)

Implementing RSA in Python from ScratchI’m sure many programmers, particularly web developers have heard of the RSA cryptography system. RSA is an asymmetric cryptography system, meaning that one key is used for encryption and the other for decryption. I’ve seen a lot of articles explaining the general principles](https://coderoasis.com/implementing-rsa-from-scratch-in-python-part-2/)  [Implementing RSA in Python from Scratch (Part 3)

Side-channel attacks The first 2 articles focused mainly on the idea and implementation of RSA. As such they left out a relevant topic in modern encryption. Side-channel attacks take advantage of data we’d usually brush off as gibberish, such as hardware sounds, electromagnetic waves and timing. Timing attacks are most](https://coderoasis.com/implementing-rsa-in-python-from-scratch-part-3/)  [Implementing RSA from Scratch in Python (Part 4)

Please note that it is essential for me to emphasize that the code and techniques presented here are intended solely for educational purposes and should never be employed in real-world applications without careful consideration and expert guidance. At the same time, understanding the principles of RSA cryptography and exploring various](https://coderoasis.com/implementing-rsa-from-scratch-in-python-part-4/) 

* * *

> Do you like what you're reading from the CoderOasis Technology Blog? We recommend reading our [***Hacktivism: Social Justice by Data Leaks and Defacements***](https://coderoasis.com/hacktivism-social-justice-by-data-leaks-and-defacements/) as your next choice.

 [Hacktivism: Social Justice by Data Leaks and Defacements

Around the end of February, a hacktivist that calls himself JaXpArO and My Little Anonymous Revival Project breached the far-right social media platform named Gab. They managed to gain seventy gigabytes of data from the backend databases. The data contained user profiles, private posts, chat messages, and more – a lot](https://coderoasis.com/hacktivism-social-justice-by-data-leaks-and-defacements/) 

* * *

> Did you know we have a [**Community Forums**](https://forums.coderoasis.com/?ref=coderoasis.com) and [**Discord Server**](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com)? which we invite everyone to join us? Want to discuss this article with other members of our community? Want to join a laid back place to chill and discuss topics like programming, cybersecurity, web development, and Linux? Consider joining us today!

 [Join the CoderOasis.com Discord Server!

CoderOasis offers technology news articles about programming, security, web development, Linux, systems admin, and more. | 112 members](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com)  [CoderOasis Forums

CoderOasis Community Forums where our members can have a place to discuss technology together and share resources with each other.](https://forums.coderoasis.com/?ref=coderoasis.com)