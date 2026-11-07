# triple-wrap

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Reverse | 150 | hibris, Clawd | reverse, encoding, easy |

## Énoncé

`chall` est un ELF x86-64 strippé qui vérifie un flag. Sa bannière affiche un flag qui est refusé, et `strings` montre surtout une longue chaîne illisible.

## Analyse

Le `.rodata` contient tout le nécessaire :

| Adresse | Contenu |
|---|---|
| `0x2008` | blob base64 `NiImIxAG...` |
| `0x2038` | leurre `NCTF{th1s_1s_n0t_th3_r34l_0ne}` |
| `0x2096` | clé `wrapkey` |

Comme le nom l'indique, le vrai flag est emballé en trois couches, à retirer dans l'ordre :

1. **Base64** : décoder le blob.
2. **XOR** avec la clé répétée `wrapkey` : on obtient `APGS{crry_o64_ebg13_k0e_ynlref}`.
3. **ROT13** : `APGS` devient `NCTF`.

## Résolution

```python
import base64, codecs, re

d = open("chall", "rb").read()
blob = re.search(rb"NiImIxAG[A-Za-z0-9+/=]+", d).group()
k = b"wrapkey"

b = base64.b64decode(blob)
s = bytes(c ^ k[i % len(k)] for i, c in enumerate(b)).decode("latin1")
print(s)
print(codecs.encode(s, "rot13"))
```

## Flag

```
NCTF{peel_b64_rot13_x0r_layers}
```

## À retenir

Empiler des encodages n'ajoute aucune sécurité si la clé est dans le binaire. La forme `APGS{` est un repère classique : c'est `NCTF{` en ROT13.
