# shell-lock

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Reverse | 86 | hibris, Clawd | reverse, shell, easy |

## Énoncé

`lock.sh` est un script auto-déchiffrant : avec le bon mot de passe il affiche le flag, sinon `access denied`.

## Analyse

Tout peut se dérouler statiquement, sans exécuter le script :

```sh
_k=5942194646755d1b504b584e
_b=PStnKhcsHwIWDS0XFgRVMwhsFEMDEQY7Aw0AAAk7Cg==
_p=$(printf '%s' "$_k" | perl -ne 'chomp;print chr(hex($1)^0x2a)while/(..)/g')
[ "$1" = "$_p" ] || { echo "access denied"; exit 1; }
```

1. Le mot de passe n'est pas secret : c'est `_k`, XOR octet par octet avec `0x2a`.
2. Le payload `_b` est décodé en base64, puis XOR avec ce même mot de passe en clé répétée.

## Résolution

```python
import base64, re

t = open("lock.sh").read()
k = re.search(r"^_k=(\w+)", t, re.M).group(1)
b = re.search(r"^_b=(\S+)", t, re.M).group(1)

pwd = bytes(x ^ 0x2a for x in bytes.fromhex(k))
ct = base64.b64decode(b)
print("mot de passe :", pwd.decode())
print(bytes(c ^ pwd[i % len(pwd)] for i, c in enumerate(ct)).decode().strip())
```

Le mot de passe est `sh3ll_w1zard`.

## Flag

```
NCTF{sh3ll_self_d3crypt_pe3led}
```

## À retenir

Le test sur le mot de passe est purement décoratif : la clé est livrée avec le chiffré. Une vraie protection dériverait la clé de la saisie de l'utilisateur (PBKDF2, scrypt) et ne stockerait qu'un sel.
