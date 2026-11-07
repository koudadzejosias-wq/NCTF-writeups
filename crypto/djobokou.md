# djobokou

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Crypto | 246 | KodjoDoDjango | crypto, medium |

## Énoncé

Un portefeuille électronique signe ses ordres de paiement en ECDSA sur secp256k1. Deux ordres ont été signés avec le même nonce `k` (les deux signatures partagent le même `r`).

## Analyse

Une signature ECDSA vérifie `s = k⁻¹(z + r·d) mod n`. Avec le même `k` :

```
s1 - s2 = k⁻¹(z1 - z2)      donc  k = (z1 - z2) / (s1 - s2) mod n
d = (s1·k - z1) / r mod n
```

`z` est le SHA-256 du message. Le résultat se vérifie sans ambiguïté en contrôlant que `d·G` redonne la clé publique `Q`.

## Résolution

```python
import hashlib

p = 115792089237316195423570985008687907853269984665640564039457584007908834671663
n = 115792089237316195423570985008687907852837564279074904382605163141518161494337
G = (55066263022277343669578718895168534326250603453777594175500187360389116729240,
     32670510020758816978083085130507043184471273380659243275938904335757337482424)
Q = (26465451447694732820083914042836869698143887696181036050684142955169784999424,
     45851276265149573712540095542872899665283249835862128912567512258826906289243)
m1, m2 = b"virement a Maman - 500 FCFA", b"virement a Papi  - 500 FCFA"
r = 34354615678816025963991240435576384569143797770590472620520679676426665325474
s1 = 101548214840016312275236379957595460827046978199001227314058520352923602465323
s2 = 56372393954078655523873920079586168249157042152285930432067707747433946407322

def add(P, R):
    if P is None: return R
    if R is None: return P
    if P[0] == R[0] and (P[1] + R[1]) % p == 0: return None
    if P == R:
        l = 3 * P[0] * P[0] * pow(2 * P[1], -1, p) % p
    else:
        l = (R[1] - P[1]) * pow(R[0] - P[0], -1, p) % p
    x = (l * l - P[0] - R[0]) % p
    return (x, (l * (P[0] - x) - P[1]) % p)

def mul(k, P):
    R = None
    while k:
        if k & 1: R = add(R, P)
        P = add(P, P)
        k >>= 1
    return R

z1, z2 = (int.from_bytes(hashlib.sha256(m).digest(), "big") for m in (m1, m2))
k = (z1 - z2) * pow(s1 - s2, -1, n) % n
d = (s1 * k - z1) * pow(r, -1, n) % n
assert mul(d, G) == Q
print("NCTF{%064x}" % d)
```

## Flag

```
NCTF{98e0c9af350a59ba3c3d4a7b320cca8981d272318762f70c9e8902574e355caf}
```

> La clé privée est certaine, car `d·G = Q`. En revanche, l'énoncé ne précise pas le format du flag : le wrapper `NCTF{<d en hexadécimal>}` a été repris du challenge GCM du même auteur et n'a pas été confirmé. La forme décimale de `d` est l'alternative.

## À retenir

Le nonce ECDSA doit être unique et imprévisible. Une seule réutilisation expose la clé privée. RFC 6979 évite ce risque en dérivant `k` de façon déterministe.
