# hull-cipher

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| PPC | 150 | hibris, Clawd | ppc, geometry, convex-hull, easy |

## Énoncé

`points.txt` liste quelques centaines de points 2D portant chacun un caractère. La plupart sont du bruit. Un petit sous-ensemble, lu dans le bon ordre géométrique, donne le flag.

## Analyse

Sur les 441 points, exactement 41 sont sur l'**enveloppe convexe**, et c'est la longueur du flag. Les points intérieurs sont le bruit.

L'algorithme de la chaîne monotone d'Andrew donne les sommets dans le sens antihoraire. Le test `cross <= 0` retire aussi les points colinéaires. En partant du point le plus à gauche, on lit :

```
meter_ccw}NCTF{convex_hull_walks_the_peri
```

La bonne séquence est là, mais elle commence au milieu. Il suffit de la faire tourner pour démarrer sur `NCTF{`. Le point `N` est `(0, -1000000)`, le plus bas de l'enveloppe.

## Résolution

```python
lignes = open("points.txt").read().split("\n")
pts = []
for l in lignes[1:]:
    if l.strip():
        a, b, c = l.split()
        pts.append((int(a), int(b), c))

def cross(o, a, b):
    return (a[0] - o[0]) * (b[1] - o[1]) - (a[1] - o[1]) * (b[0] - o[0])

def chaine(P):
    h = []
    for p in P:
        while len(h) >= 2 and cross(h[-2], h[-1], p) <= 0:
            h.pop()
        h.append(p)
    return h[:-1]

P = sorted(pts)
s = "".join(p[2] for p in chaine(P) + chaine(P[::-1]))
i = s.index("NCTF{")
print(s[i:] + s[:i])
```

## Flag

```
NCTF{convex_hull_walks_the_perimeter_ccw}
```

## À retenir

Le fait que le nombre de sommets (41) égale la longueur du flag confirme la piste, et le flag lui-même indique le sens de parcours.
