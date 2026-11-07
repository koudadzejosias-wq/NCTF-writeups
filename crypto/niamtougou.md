# niamtougou

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Crypto | 249 | KodjoDoDjango | crypto, medium |

## Énoncé

Un générateur a produit 624 sorties de 32 bits avec le PRNG de Python, puis a chiffré le flag par XOR avec la clé tirée juste après.

## Analyse

`random` utilise Mersenne Twister (MT19937), dont l'état interne est fait de 624 mots de 32 bits. Chaque sortie est un mot d'état passé dans une fonction de *tempering* entièrement inversible. Avec 624 sorties consécutives, on reconstruit donc tout l'état :

1. inverser le tempering de chaque sortie ;
2. injecter l'état dans `random.Random` avec `setstate` et l'index 624 ;
3. tirer la suite du flux.

Le format de la clé n'était pas précisé. Plusieurs variantes ont été testées, et le bon format est une suite de `getrandbits(32)` sérialisés en big-endian.

## Résolution

```python
import random, re

t = open("challenge.txt").read()
sorties = [int(x) for x in re.findall(r"^o\d+ = (\d+)", t, re.M)]
enc = bytes.fromhex(re.search(r"enc = ([0-9a-f]+)", t).group(1))
assert len(sorties) == 624

def inv_droite(y, s):
    x = y
    for _ in range(32 // s + 1):
        x = y ^ (x >> s)
    return x

def inv_gauche(y, s, masque):
    x = y
    for _ in range(32 // s + 1):
        x = y ^ ((x << s) & masque & 0xffffffff)
    return x

def untemper(y):
    y = inv_droite(y, 18)
    y = inv_gauche(y, 15, 0xefc60000)
    y = inv_gauche(y, 7, 0x9d2c5680)
    return inv_droite(y, 11)

r = random.Random()
r.setstate((3, tuple(untemper(v) for v in sorties) + (624,), None))

flux = b""
while len(flux) < len(enc):
    flux += r.getrandbits(32).to_bytes(4, "big")
print(bytes(a ^ b for a, b in zip(enc, flux)).decode())
```

## Flag

```
NCTF{N14M70UG0U_MT19937_5T4T3_R3C0V3RY_483726591}
```

## À retenir

Mersenne Twister n'est pas un générateur cryptographique. Pour une clé, il faut utiliser `secrets` ou `os.urandom`.
