# bassar

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Crypto | 249 | KodjoDoDjango | crypto, medium |

## Énoncé

Un même message a été chiffré en RSA pour trois destinataires, avec trois modules différents et le même exposant `e = 3`.

## Analyse

C'est l'attaque de diffusion de Håstad. On a :

```
c1 = m³ mod n1
c2 = m³ mod n2
c3 = m³ mod n3
```

1. Les modules sont premiers entre eux deux à deux (les `gcd` valent 1), donc le théorème des restes chinois s'applique.
2. Le CRT donne `x ≡ m³ mod (n1·n2·n3)`.
3. Comme `m < ni`, on a `m³ < n1·n2·n3` : `x` est exactement `m³`, sans réduction modulaire.
4. Il suffit alors d'une racine cubique entière.

## Résolution

```python
import re
from math import gcd

t = open("challenge.txt").read()
v = {k + i: int(x) for k, i, x in re.findall(r"^([nc])(\d) = (\d+)", t, re.M)}
n = [v["n1"], v["n2"], v["n3"]]
c = [v["c1"], v["c2"], v["c3"]]
assert gcd(n[0], n[1]) == gcd(n[0], n[2]) == gcd(n[1], n[2]) == 1

N = n[0] * n[1] * n[2]
x = sum(ci * (N // ni) * pow(N // ni, -1, ni) for ci, ni in zip(c, n)) % N

lo, hi = 0, 1 << (N.bit_length() // 3 + 2)
while lo < hi:
    m = (lo + hi) // 2
    if m ** 3 < x:
        lo = m + 1
    else:
        hi = m
assert lo ** 3 == x
print(lo.to_bytes((lo.bit_length() + 7) // 8, "big").decode())
```

## Flag

```
NCTF{B4S54R_H4574D_BR04DC457_4TT4CK_2094837411}
```

## À retenir

Sans padding aléatoire (OAEP), un petit exposant et un même message envoyé à `e` destinataires suffisent à retrouver le clair.
