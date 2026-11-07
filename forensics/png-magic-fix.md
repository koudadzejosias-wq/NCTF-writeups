# png-magic-fix

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Forensics | 97 | hibris, Clawd | forensics, file-carving, png, beginner |

## Énoncé

`evidence.png` ne s'ouvre plus. Les pixels semblent intacts, seul le début du fichier paraît abîmé. La note de l'analyste, dans les métadonnées, est le flag.

## Analyse

Le hexdump montre le problème immédiatement :

```
00000000: 0000 0000 0000 0000 0000 000d 4948 4452  ............IHDR
```

Les 8 octets de la signature PNG ont été remplacés par des zéros, alors que le chunk `IHDR` est intact. Il suffit de réécrire la signature :

```
89 50 4E 47 0D 0A 1A 0A
```

Après réparation, les CRC des quatre chunks sont valides, ce qui confirme qu'aucun autre octet n'a été touché. La note est dans un chunk `zTXt` de mot-clé `Comment`. Ce texte est compressé en zlib, d'où son absence dans le hexdump brut.

## Résolution

```python
import struct, zlib

d = bytearray(open("evidence.png", "rb").read())
d[:8] = b"\x89PNG\r\n\x1a\n"
open("evidence_fixed.png", "wb").write(d)

i = 8
while i < len(d):
    n = struct.unpack(">I", d[i:i + 4])[0]
    t, b = bytes(d[i + 4:i + 8]), bytes(d[i + 8:i + 8 + n])
    if t == b"zTXt":
        cle, reste = b.split(b"\0", 1)
        print(cle.decode(), "=", zlib.decompress(reste[1:]).decode())
    i += 12 + n
```

## Flag

```
NCTF{png_m4gic_byt3s_r3st0r3d}
```

## À retenir

Les visionneuses identifient un fichier par ses *magic bytes*. Ceux-ci se corrigent en quelques octets, et les CRC des chunks permettent de valider la réparation.
