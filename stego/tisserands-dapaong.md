# tisserands-dapaong

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Stégano | 250 | Algorithme123 | stego, easy-medium, togo, dapaong |

## Énoncé

Pour un festival fictif à Dapaong, on reçoit une affiche officielle et un guide des motifs. Il faut retrouver le message tissé destiné aux partenaires.

## Analyse

Le flux texte de `guide-motifs.pdf` et les métadonnées de `affiche.png` donnent la recette :

- le « fil rouge » ou « fibre primaire » désigne le **canal rouge** ;
- la lecture se fait en deux temps : longueur `uint32` big-endian, puis motifs UTF-8 ;
- les consonnes et les chiffres sont écrits avec leur glyphe latin (`n->N`, `s->S`, `r->R`, `b->B`) ;
- les voyelles ont un glyphe propre, visible uniquement sur l'affiche.

L'extraction du LSB rouge, ligne par ligne sur 900×1200 pixels, donne une longueur de 39, qui correspond exactement au contenu :

```
NCTF{D4P40NG_T1SS3R4NDS_29549DB55○2C}
```

Un balayage des 8 plans de bits des 3 canaux, en ligne et en colonne, confirme que c'est le seul message présent.

## Résolution

```python
from PIL import Image

im = Image.open("affiche.png").convert("RGB")
w, h = im.size
p = im.load()
b = "".join(str(p[x, y][0] & 1) for y in range(h) for x in range(w))
n = int(b[:32], 2)
brut = bytes(int(b[i:i + 8], 2) for i in range(32, 32 + n * 8, 8)).decode()
print(brut)
print(brut[:5] + brut[5:-1].replace("○", "e").lower() + brut[-1])
```

Reste à passer des motifs au flag : la chaîne tissée est une **représentation**, il faut la remettre en minuscules et remplacer `○`.

Ce dernier point n'a pas pu être déduit. Le guide indique « Hex a reste une voyelle (glyphe ○ sur l'affiche) », ce qui a d'abord fait choisir `a`, et ce choix était faux. Dans un suffixe hexadécimal, les seules voyelles possibles sont `a` et `e`, et la légende de l'affiche ne permet pas de trancher car ses cinq glyphes sont des carrés identiques au pixel près. C'est en soumettant la variante `e` que le flag a été validé par la plateforme.

## Flag

```
NCTF{d4p40ng_t1ss3r4nds_29549db55e2c}
```

## À retenir

Il faut bien distinguer le message extrait de son encodage. Ici, la longueur `uint32` valide l'extraction, mais pas l'interprétation des glyphes.
