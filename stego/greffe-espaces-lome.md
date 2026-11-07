# greffe-espaces-lome

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Stégano | 175 | Algorithme123 | stego, easy, whitespace, togo |

## Énoncé

Le greffe du tribunal de Lomé (fictif) publie un arrêté en texte brut, et quelque chose cloche dans la mise en page.

## Analyse

D'après `note-mise-en-page.txt`, chaque ligne de `arrete-greffe-lome.txt` se termine par un caractère invisible significatif :

- espace : `0` ;
- tabulation : `1`.

Les 248 lignes donnent 248 bits, soit 31 octets lus de haut en bas, MSB d'abord. Les premiers bits donnent `01001110`, soit `N`.

Le texte `# NCTF{brouillon_greffe}` visible en clair à la ligne 22 est un leurre.

## Résolution

```python
import re

bits = ""
for ligne in open("arrete-greffe-lome.txt", encoding="utf-8", newline="").read().split("\n"):
    m = re.search(r"[ \t]+$", ligne.rstrip("\r"))
    if m:
        bits += "".join("1" if c == "\t" else "0" for c in m.group())

print(bytes(int(bits[i:i + 8], 2) for i in range(0, len(bits) // 8 * 8, 8)).decode())
```

## Flag

```
NCTF{gr3ff3_sp4c3_43e9df0e3a4e}
```

## À retenir

Un flag lisible en clair dans un fichier de stégano est presque toujours un leurre. Les espaces de fin de ligne sont un canal discret, que `cat -A` rend visible.
