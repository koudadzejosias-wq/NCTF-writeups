# sakpata

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Crypto | 400 | KodjoDoDjango | crypto, hard |

## Énoncé

Chaque bit du flag choisit un poids parmi 88 (bit `i` à 1 : poids `i` dans la somme), et seule la somme `target` est donnée. L'énoncé oriente vers un subset-sum de basse densité résolu par réseau (Coster et al., LLL/BKZ).

## Analyse

Le flag fait 88 bits, soit 11 octets : `NCTF{` + 5 caractères + `}`. Ce format réduit beaucoup le problème, et l'attaque par réseau devient inutile :

- 6 octets sur 11 sont connus, ils représentent 48 bits fixés ;
- les 5 octets inconnus sont de l'ASCII, donc leur bit de poids fort vaut 0 ;
- il reste 35 bits inconnus.

Un *meet-in-the-middle* suffit alors : on stocke les 2¹⁷ sommes de la première moitié, puis on cherche le complément parmi les 2¹⁸ sommes de la seconde.

La convention de bits n'étant pas donnée, quatre ordres ont été testés. Un seul produit une solution : le bit `i = 0` est le bit de poids fort du premier octet.

## Résolution

```python
import ast, re

t = open("challenge.txt").read()
w = ast.literal_eval(re.search(r"weights = (\[.*?\])", t, re.S).group(1))
T = int(re.search(r"target = (\d+)", t).group(1))
poids = lambda octet, bit: w[octet * 8 + (7 - bit)]

connu = {i: v for i, v in enumerate(b"NCTF{")}
connu[10] = ord("}")
reste = T - sum(poids(o, b) for o, v in connu.items() for b in range(8) if v >> b & 1)

inconnus = [(o, b) for o in range(5, 10) for b in range(7)]
a, c = inconnus[:17], inconnus[17:]

def sommes(bits):
    s = [0]
    for o, b in bits:
        x = poids(o, b)
        s += [v + x for v in s]
    return s

table = {v: m for m, v in enumerate(sommes(a))}
for m, v in enumerate(sommes(c)):
    if reste - v in table:
        f = bytearray(11)
        f[:5], f[10] = b"NCTF{", ord("}")
        for bits, masque in ((a, table[reste - v]), (c, m)):
            for j, (o, b) in enumerate(bits):
                if masque >> j & 1:
                    f[o] |= 1 << b
        assert sum(poids(o, b) for o in range(11) for b in range(8) if f[o] >> b & 1) == T
        print(f.decode())
```

## Flag

```
NCTF{KN4PS}
```

## À retenir

Un format de flag connu est de l'information. Ici, il ramène un problème de 88 bits à 35 bits, à la portée d'un simple meet-in-the-middle.
