# lsb-bmp

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Stégano | 89 | hibris, Clawd | stego, lsb, bmp, beginner |

## Énoncé

Une carte postale `postcard.bmp` ne montre qu'un dégradé, mais elle cache un message.

## Analyse

Le fichier est un BMP 96×64 en 24 bits, avec les pixels à l'offset `0x36`. Une recherche exhaustive (ordre des lignes, canaux, ordre des bits) ne sort qu'une combinaison lisible :

| Propriété | Valeur |
|---|---|
| Support | LSB du canal **bleu** uniquement (premier octet de chaque triplet BGR) |
| Ordre des bits | MSB d'abord, 8 LSB par caractère |
| Départ | Première ligne du fichier, donc la ligne du **bas** de l'image |

Les canaux vert et rouge sont des leurres : leurs LSB forment des motifs réguliers (`0x00/0xFF`, `0x55/0xAA`). Une extraction RGB entrelacée ne donne donc que du bruit.

## Résolution

```python
import struct

d = open("postcard.bmp", "rb").read()
off = struct.unpack_from("<I", d, 10)[0]
w, h = struct.unpack_from("<ii", d, 18)
stride = (w * 24 + 31) // 32 * 4

bits = [d[off + r * stride + c * 3] & 1 for r in range(h) for c in range(w)]
out = bytes(int("".join(map(str, bits[i:i + 8])), 2) for i in range(0, len(bits) - 7, 8))
print(out.split(b"\0")[0].decode())
```

## Flag

```
NCTF{lsb_blue_channel_whispers}
```

## À retenir

Il faut tester les canaux séparément : une lecture entrelacée masque un message porté par un seul canal. Le BMP stocke aussi ses lignes de bas en haut et ses pixels en BGR.
