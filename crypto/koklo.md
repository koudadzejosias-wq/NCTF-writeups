# koklo

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Crypto | 250 | KodjoDoDjango | crypto, medium |

## Énoncé

Le service bancaire Koklo chiffre ses virements en AES-GCM mais a réutilisé le même nonce pour deux messages. Il faut forger un tag valide pour `PAY:FROM=ADMIN:9`. Le fichier du challenge précise que le flag est le tag forgé, en hexadécimal, dans `NCTF{...}`.

## Analyse

Les trois messages font un seul bloc de 16 octets, sans données associées. Avec un même nonce, le flux CTR et le masque du tag sont identiques. Le tag vaut alors :

```
t = c·H² ⊕ L·H ⊕ E(J0)
```

`L` (bloc de longueur) et `E(J0)` sont communs aux trois messages. En faisant le XOR de deux tags, ils disparaissent :

```
t1 ⊕ t2 = (c1 ⊕ c2)·H²      donc  H² = (t1 ⊕ t2) / (c1 ⊕ c2)
```

La forge ne demande ni la clé AES ni même `H` :

```
c3 = c1 ⊕ p1 ⊕ p3
t3 = t1 ⊕ (c1 ⊕ c3)·H²
```

Toutes les multiplications ont lieu dans GF(2¹²⁸) avec la convention GCM (bits réfléchis). La forge a aussi été validée contre une vraie implémentation AES-GCM : le chiffré et le tag calculés étaient identiques.

## Résolution

```python
R = 0xE1 << 120
def mul(x, y):
    z, v = 0, x
    for i in range(127, -1, -1):
        if (y >> i) & 1:
            z ^= v
        v = (v >> 1) ^ (R if v & 1 else 0)
    return z

def gpow(x, e):
    r = 1 << 127
    while e:
        if e & 1:
            r = mul(r, x)
        x = mul(x, x)
        e >>= 1
    return r

inv = lambda x: gpow(x, (1 << 128) - 2)
I = lambda b: int.from_bytes(b, "big")

p1, p3 = b"PAY:FROM=GUEST:1", b"PAY:FROM=ADMIN:9"
c1 = bytes.fromhex("8c848e7c20acc8f8856843b73454bbfa")
t1 = bytes.fromhex("24e6048ed904bed915a6c49a03906c5c")
c2 = bytes.fromhex("8c848e7c20acc8f8856843b73454bbf9")
t2 = bytes.fromhex("596ec53b597c4c10ac0010a3a77c9402")

h2 = mul(I(t1) ^ I(t2), inv(I(c1) ^ I(c2)))
c3 = bytes(a ^ b ^ d for a, b, d in zip(c1, p1, p3))
t3 = I(t1) ^ mul(I(c1) ^ I(c3), h2)
print("c3 =", c3.hex())
print("NCTF{%032x}" % t3)
```

## Flag

```
NCTF{7736a42084756aed78022905afeca950}
```

## À retenir

En GCM, réutiliser un nonce ne casse pas seulement la confidentialité : cela casse aussi l'authenticité, car la clé d'authentification devient calculable.
