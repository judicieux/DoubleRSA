# DoubleRSA
On se retrouve pour discuter un challenge RSA assez particulier.
## Chiffrement
```py
from Crypto.Util.number import *
import gmpy2

flag = "XXXXXXXX"

p = getPrime(512)
q = getPrime(512)
r = getPrime(512)
n1 = p * q
n2 = p * q * r

e1 = getPrime(20)
e2 = int(gmpy2.next_prime(e1))

m = bytes_to_long(flag)
c1 = pow(m, e1, n1)
c2 = pow(m, e2, n2)

print("n1 = {}".format(n1))
print("n2 = {}".format(n2))
print("e1 = {}".format(e1))
print("e2 = {}".format(e2))
print("c1 = {}".format(c1))
print("c2 = {}".format(c2))
```
## Manuel
```p,q,r```: Three very large primes<br/>
```n1```: First modulus<br/>
```n2```: Second modulus<br/>
```e1```: First key exponent<br/>
```e2```: Second key exponent<br/>
```c1```: First ciphertext<br/>
```c2```: Second ciphertext<br/>
## Soluce
En effet, on a deux modulus dont un qui est composé de 3 nombres premiers.<br/>
On a aussi 2 exponents allignés, pas compliqué à comprendre.<br/>
La première chose qu'on peut se demander: Comment recover ```r``` et sa clé privée à partir des valeurs qu'on a?<br/>
Et bien figurez vous que c'est pas si compliqué que ç'en a l'air.<br/>
### Test
Essayons de reproduire la multiplication des nombres premiers avec des petits nombres.<br/>
```py
>>> n1 = 5 * 4
>>> n2 = 5 * 4 * 9
>>> c = n2 // n1
>>> print(c)
9
```
