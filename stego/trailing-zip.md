# trailing-zip

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Stégano | 100 | hibris, Clawd | stego, polyglot, appended-data, beginner |

## Énoncé

`banner.bmp` s'ouvre normalement, mais le fichier est trop gros pour une si petite image.

## Analyse

Il suffit de comparer la taille attendue à la taille réelle :

| Élément | Octets |
|---|---:|
| En-têtes | 54 |
| Pixels (80×80, 24 bits) | 19 200 |
| `bfSize` déclaré | 19 254 |
| Taille réelle | 19 416 |
| **Surplus** | **162** |

Le surplus commence par `PK\x03\x04` juste après les pixels : c'est une archive ZIP complète. Les visionneuses s'arrêtent à la fin des pixels et ne la voient jamais. Comme les lecteurs ZIP cherchent la fin de l'archive en partant de la fin du fichier, le fichier est un vrai polyglotte.

## Résolution

```python
import io, struct, zipfile

b = open("banner.bmp", "rb").read()
taille = struct.unpack_from("<I", b, 2)[0]
z = zipfile.ZipFile(io.BytesIO(b[taille:]))
for i in z.infolist():
    print(i.filename, z.read(i).decode().strip())
```

L'archive contient une seule entrée non chiffrée, `secret/flag.txt`.

## Flag

```
NCTF{zip_hides_past_the_pixels}
```

## À retenir

Comparer la taille déclarée à la taille réelle est un réflexe simple et efficace. `binwalk` automatise cette recherche.
