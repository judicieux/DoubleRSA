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
print("c1 = {}".format(c1))
print("c2 = {}".format(c2))

"""
n1 = 154935565403645385721284171603184453142670245088181207053410793795106461397612583512380166397225422847494826531450774742551645956962389910358184436862682282246827141646775401052637574142707206015370634903977975983406238952485342823071915176992003895698440907211930889860062847100847368718308879607755780566671
n2 = 1547065595046287231222849370165620395247952974041526328065680194977932152638377271582389978235991816932552754512430114535293810382643386407557754039452917684442506460545446175177440740669282644427762822791495694747715075106206795255985503206131281937023827796976496377661801118129564774626293720203160482334165894311689158006726240034600702778424343751200686210379405029286656771861681691238856769453909401838766947392548275306125960286228437273743154506313483033
e1 = 650759
c1 = 60646693476181615208969784902507501480144072798809811487518819933745699048772562669352286768329123406759094534836136640006440458000736152168558853670124686784510997573760492611561620377358642944325457496377248050342354655057861500429907688003995830208686937093892589026743626582601301867364160369366141430651
c2 = 1033821916864288405257760878956731738683606263416675389059231051221272529939579306028448602350916716271333172356515815315550354591279219800724167479456674588576772141330447965531348639758031656868611255841879238527614860482508464163835552645371288193717094167203497742596211387333865395973528821238352848778497063120938963049283260291084316482234552166859165582435308100737661878880118941729864571903058627936286399502495427580524917209678524311900619126875756054
"""
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
En effet, on a deux modulos dont un qui est composé de 3 nombres premiers.<br/>
On a aussi 2 exponents allignés, facile à distinguer.<br/>
La première chose qu'on peut se demander c'est: Comment recover ```r``` et sa clé privée à partir des valeurs qu'on a?<br/>
Et bien figurez vous que c'est pas si compliqué que ç'en a l'air.<br/>
### Test
Essayons de reproduire la composition des modulos avec des petits nombres pour mieux reconnaître ```r``` et s'assurer de la formula.<br/>
```py
>>> n1 = 5 * 4
>>> n2 = 5 * 4 * 9
>>> r = n2 // n1
>>> print(r)
9
```
On peut décomposer ```r``` de ```n2``` en divisant ```n2``` par ```n1```.<br/>
```py
>>> n1 = 154935565403645385721284171603184453142670245088181207053410793795106461397612583512380166397225422847494826531450774742551645956962389910358184436862682282246827141646775401052637574142707206015370634903977975983406238952485342823071915176992003895698440907211930889860062847100847368718308879607755780566671
>>> n2 = 1547065595046287231222849370165620395247952974041526328065680194977932152638377271582389978235991816932552754512430114535293810382643386407557754039452917684442506460545446175177440740669282644427762822791495694747715075106206795255985503206131281937023827796976496377661801118129564774626293720203160482334165894311689158006726240034600702778424343751200686210379405029286656771861681691238856769453909401838766947392548275306125960286228437273743154506313483033
>>> r = n2 // n1
>>> r
9985219281420631491950957851274022801091454644758524755532672893128337056039820257283811157323428103614562402318170808820186779244337280644056842166257623
```
On a plus qu'à recover la clé privée de ```r```.<br/>
Pour ce faire on va calculer l'inverse modulaire de ```e2```:<br/>
```py
>>> from gmpy2 import next_prime
>>> e1 = 650759
>>> e2 = int(next_prime(e1))
>>> r = 9985219281420631491950957851274022801091454644758524755532672893128337056039820257283811157323428103614562402318170808820186779244337280644056842166257623
>>> d = e2**-1%(r-1)
>>> d
1956763019452193466224973226575382215213864900450580760830497181057219839150763733460721502148999119138446064654196192974704936830407170049672056117278595
```
### Flag
On peut déchiffrer le flag.<br/>
```py
>>> from Crypto.Util.number import long_to_bytes
>>> d = 1956763019452193466224973226575382215213864900450580760830497181057219839150763733460721502148999119138446064654196192974704936830407170049672056117278595
>>> c2 = 1033821916864288405257760878956731738683606263416675389059231051221272529939579306028448602350916716271333172356515815315550354591279219800724167479456674588576772141330447965531348639758031656868611255841879238527614860482508464163835552645371288193717094167203497742596211387333865395973528821238352848778497063120938963049283260291084316482234552166859165582435308100737661878880118941729864571903058627936286399502495427580524917209678524311900619126875756054
>>> r = 9985219281420631491950957851274022801091454644758524755532672893128337056039820257283811157323428103614562402318170808820186779244337280644056842166257623
>>> flag = pow(c2%r, d, r)
>>> long_to_bytes(flag)
PlayCTF{Cl4Ss1C_CrYpT0_M3tH0d}
```
