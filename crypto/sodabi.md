# sodabi

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Crypto | 98 | KodjoDoDjango | crypto, medium |

## Énoncé

Deux messages ont été chiffrés avec le même masque jetable. Le premier est le flag, le second est un texte public fourni avec les chiffrés.

## Analyse

C'est un *two-time pad* :

```
c1 = flag  ⊕ K
c2 = known ⊕ K
```

Le texte clair `known` donne directement le flux de clé, et donc le flag :

```
K    = c2 ⊕ known
flag = c1 ⊕ K = c1 ⊕ c2 ⊕ known
```

`c1` fait 32 octets et `c2` en fait 74 : les 32 premiers octets du flux suffisent.

## Résolution

```python
c1 = bytes.fromhex("8ffff4c3d0ba362e540bb291cd1af41205cfb7c024dfa86fa3d01c7e086a8e51")
c2 = bytes.fromhex("83d3ceefc49c7446402ae6adef09c83e23dc979b37edfc2ee982432f1f2fcd4e91f47ce4c557f1c6eeb622c686f020ef1f463cc80b42512244cc7c4ee1b695ca453c846a4b78d6583c4c")
known = b"Bonjour, ceci est un message public pour verifier la cle XOR du challenge."

cle = bytes(a ^ b for a, b in zip(c2, known))
print(bytes(a ^ b for a, b in zip(c1, cle)).decode())
```

## Flag

```
NCTF{S0D4B1_K3Y_R3U53_129384756}
```

## À retenir

Un masque jetable n'est sûr que s'il sert une seule fois. Dès qu'il est réutilisé, `c1 ⊕ c2` fait disparaître la clé.
