# png-text-chunk

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Stégano | 150 | hibris, Clawd | stego, metadata, png, easy |

## Énoncé

Le fichier `sunset.png` a des métadonnées bavardes. L'un des champs est un leurre inoffensif.

## Analyse

Un parcours des chunks PNG donne deux `tEXt` :

| Mot-clé | Valeur |
|---|---|
| `Description` | `a lovely generated sunset, nothing to see` (le leurre) |
| `Signature` | `TkNURnt0ZXh0X2NodW5rX2NhcnJpZXNfc2VjcmV0c30=` |

La `Signature` est du base64 : `TkNURn` décode vers `NCTF`.

## Résolution

```python
import base64, struct

d = open("sunset.png", "rb").read()
o = 8
while o + 8 <= len(d):
    n, t = struct.unpack(">I4s", d[o:o + 8])
    b = d[o + 8:o + 8 + n]
    o += 12 + n
    if t == b"tEXt":
        cle, val = b.split(b"\0", 1)
        print(cle.decode(), "=", val.decode())
        if cle == b"Signature":
            print(base64.b64decode(val).decode())
```

## Flag

```
NCTF{text_chunk_carries_secrets}
```

## À retenir

Les métadonnées font partie du fichier : `exiftool` ou un parcours des chunks est à faire avant toute analyse plus lourde.
