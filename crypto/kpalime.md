# kpalime

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Crypto | 400 | KodjoDoDjango | crypto, hard |

## Énoncé

Un serveur chiffre des jetons de la forme `préfixe_connu || secret` avec `e = 3`. Le secret est le flag et fait au plus 128 bits.

## Analyse

Le fichier donne `n` (1023 bits), `c`, le préfixe et `suffix_bits = 127`. Le chiffré vérifie :

```
c = (prefixe·2^127 + x)³ mod n,   avec x < 2^127
```

C'est la situation de la méthode de Coppersmith : `x` est une petite racine de `f(x) = (M + x)³ - c mod n`, et `2^127` est largement sous la borne `n^(1/3)`. Pas besoin de factoriser `n`.

Avec la construction de Howgrave-Graham (`m = 1`), on réduit par LLL le réseau formé par `n`, `n·xX`, `n·(xX)²` et `f(xX)`, avec `X = 2^127`. Un vecteur court donne un polynôme `h` à coefficients entiers qui a la même racine **sur les entiers**. Il ne reste qu'à chercher cette racine.

Le LLL est implémenté à la main avec des fractions exactes : en dimension 4, c'est instantané.

## Résolution

```python
import re
from fractions import Fraction as Fr

t = open("challenge.txt", encoding="utf-8", newline="").read()
n = int(re.search(r"^n = (\d+)", t, re.M).group(1))
c = int(re.search(r"^c = (\d+)", t, re.M).group(1))
prefixe = re.search(r"^prefix = (.*?)\r?$", t, re.M).group(1).encode()
b = int(re.search(r"^suffix_bits = (\d+)", t, re.M).group(1))
M, X = int.from_bytes(prefixe, "big") << b, 1 << b

def dot(u, v): return sum(a * e for a, e in zip(u, v))

def lll(B, delta=Fr(3, 4)):
    B = [l[:] for l in B]
    k = len(B)
    def gs():
        O, Mu = [], [[Fr(0)] * k for _ in range(k)]
        for i in range(k):
            v = [Fr(a) for a in B[i]]
            for j in range(i):
                Mu[i][j] = dot(B[i], O[j]) / dot(O[j], O[j])
                v = [a - Mu[i][j] * e for a, e in zip(v, O[j])]
            O.append(v)
        return O, Mu
    O, Mu = gs()
    i = 1
    while i < k:
        for j in range(i - 1, -1, -1):
            q = round(Mu[i][j])
            if q:
                B[i] = [a - q * e for a, e in zip(B[i], B[j])]
                O, Mu = gs()
        if dot(O[i], O[i]) >= (delta - Mu[i][i - 1] ** 2) * dot(O[i - 1], O[i - 1]):
            i += 1
        else:
            B[i], B[i - 1] = B[i - 1], B[i]
            O, Mu = gs()
            i = max(i - 1, 1)
    return B

def ev(p, x):
    r = 0
    for a in reversed(p):
        r = r * x + a
    return r

def racines(p, lo, hi):
    while p and p[-1] == 0:
        p = p[:-1]
    if len(p) < 2:
        return set()
    bornes = sorted(racines([i * p[i] for i in range(1, len(p))], lo, hi) | {lo, hi})
    res = set()
    for a, e in zip(bornes, bornes[1:]):
        fa, fe = ev(p, a), ev(p, e)
        if (fa < 0) != (fe < 0):
            while e - a > 1:
                m = (a + e) // 2
                if (ev(p, m) < 0) == (fa < 0):
                    a = m
                else:
                    e = m
        res |= {a, e}
    return res

f = [(M ** 3 - c) % n, 3 * M ** 2 % n, 3 * M % n, 1]
base = [
    [n, 0, 0, 0],
    [0, n * X, 0, 0],
    [0, 0, n * X ** 2, 0],
    [f[0], f[1] * X, f[2] * X ** 2, X ** 3],
]
for v in lll(base):
    h = [a // X ** i for i, a in enumerate(v)]
    for x in racines(h, 0, X):
        if pow(M + x, 3, n) == c:
            print(x.to_bytes((x.bit_length() + 7) // 8, "big").decode())
            raise SystemExit
```

## Flag

```
NCTF{SM4LL_R00T}
```

## À retenir

Un padding fixe et connu ne protège pas RSA avec un petit exposant : si la partie inconnue est assez courte, elle se retrouve par réduction de réseau sans toucher à la factorisation.
