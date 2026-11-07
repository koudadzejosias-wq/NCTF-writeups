# intel-hex

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Hardware | 100 | hibris, Clawd | hardware, intel-hex, memory-dump, beginner |

## Énoncé

`image.hex` est un dump mémoire de microcontrôleur au format texte utilisé par les outils de flashage. Une fois la mémoire reconstruite, le flag se trouve à l'offset `0x40`, mais les octets ont subi une transformation simple et réversible.

## Analyse

Le format est de l'**Intel HEX**. Chaque ligne `:10xxxx00...` contient 16 octets à l'adresse `xxxx`, et `:00000001FF` marque la fin. Le fichier couvre `0x00` à `0xFF`, sans adresse étendue, et aucun checksum n'est faux.

À l'offset `0x40`, les données commencent par `E4 34 45 64 B7 96 E6 47`. Le format de flag connu donne la transformation :

| Octet stocké | Après échange | Caractère |
|---|---|---|
| `E4` | `4E` | `N` |
| `34` | `43` | `C` |
| `45` | `54` | `T` |
| `64` | `46` | `F` |
| `B7` | `7B` | `{` |

Les deux quartets de chaque octet sont inversés (*nibble swap*), ce qui équivaut à une rotation de 4 bits. Le flag se termine à `0x61`, où `D7` devient `7D`, soit `}`.

## Résolution

Les trois lignes utiles du fichier suffisent pour l'exemple :

```python
image = """
:10004000E4344564B796E64756C6F5865687F5F616
:100050006666375647F5E6962626C656F53777166E
:1000600007D7F1D02BF2D5446F467978735ADD6CFF
"""

mem = {}
for ligne in image.split():
    r = bytes.fromhex(ligne[1:])
    assert sum(r) & 0xFF == 0
    n, adresse, type_, data = r[0], r[1] << 8 | r[2], r[3], r[4:4 + r[0]]
    if type_ == 0:
        for i, b in enumerate(data):
            mem[adresse + i] = b

swap = lambda b: ((b << 4) | (b >> 4)) & 0xFF
print(bytes(swap(mem[a]) for a in range(0x40, 0x62)).decode())
```

## Flag

```
NCTF{intel_hex_offset_nibble_swap}
```

## À retenir

Intel HEX porte ses adresses et ses checksums : il faut reconstruire la mémoire à partir des enregistrements, et non lire le fichier comme du texte. Un préfixe connu suffit ensuite à identifier une transformation simple.
