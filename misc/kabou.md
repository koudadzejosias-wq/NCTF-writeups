# kabou

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Misc | 100 | KodjoDoDjango | misc, easy |

## Énoncé

Une image `image.png` cache un secret.

## Analyse

L'image fait 400×300 en RGB. Elle n'a aucun chunk de texte (`tEXt`, `zTXt`, `eXIf`) et rien après `IEND`. Reste la piste LSB.

Parmi les combinaisons testées, une seule donne un résultat :

- 1 bit par canal, dans l'ordre R, G, B ;
- pixels lus ligne par ligne ;
- bits regroupés par 8, MSB d'abord.

Le flag occupe les premiers octets du flux et se termine par un octet nul. Le reste est du bruit, ce qui explique l'aspect de l'image.

## Résolution

```python
from PIL import Image

im = Image.open("image.png").convert("RGB")
px = list(im.getdata())[:120]
bits = [c & 1 for p in px for c in p]
out = bytes(int("".join(map(str, bits[i:i + 8])), 2) for i in range(0, len(bits) - 7, 8))
print(out.split(b"\0")[0].decode())
```

## Flag

```
NCTF{K4B0U_L5B_5T3G4N0_PN66_1192837465}
```

## À retenir

Le format PNG conserve les pixels sans perte, ce qui en fait un support naturel pour la stégano LSB. `zsteg` automatise ce type de test.
