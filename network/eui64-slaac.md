# eui64-slaac

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Réseau | 100 | hibris, Clawd | networking, ipv6, eui64, beginner |

## Énoncé

Un inventaire SLAAC (`hosts.json`) liste 29 interfaces avec leur MAC, un préfixe /64 commun et un tag d'un caractère. Il faut reconstruire les adresses IPv6, ordonner les hôtes et lire les tags.

## Analyse

Pour obtenir l'identifiant d'interface EUI-64 à partir d'une MAC :

1. couper la MAC en deux et insérer `ff:fe` au milieu ;
2. inverser le bit universel/local (bit 1 du premier octet).

Exemple :

```
0c:19:c2:16:06:6b  ->  0e:19:c2:ff:fe:16:06:6b
adresse : 2001:db8:ac1d:64:e19:c2ff:fe16:66b
```

L'adresse finale est le préfixe `2001:db8:ac1d:64::` combiné à cet identifiant. Il reste à trier les 29 hôtes par adresse numérique et à lire les tags.

L'ordre obtenu correspond à `node-00` ... `node-28`, ce qui confirme le tri.

## Résolution

```python
import ipaddress, json

data = json.load(open("hosts.json"))
base = int(ipaddress.IPv6Address(data["prefix"].split("/")[0]))

lignes = []
for h in data["hosts"]:
    o = [int(x, 16) for x in h["mac"].split(":")]
    o[0] ^= 0x02
    iid = int.from_bytes(bytes(o[:3] + [0xFF, 0xFE] + o[3:]), "big")
    lignes.append((base | iid, h["tag"]))

print("".join(tag for _, tag in sorted(lignes)))
```

## Flag

```
NCTF{eui64_slaac_ll_bit_flip}
```

## À retenir

EUI-64 expose la MAC dans l'adresse IPv6, ce qui permet de suivre un appareil d'un réseau à l'autre. C'est la raison d'être des adresses de confidentialité (RFC 8981).
