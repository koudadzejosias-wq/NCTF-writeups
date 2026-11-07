# bgp-bestpath

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Réseau | 150 | hibris, Clawd | networking, bgp, routing, easy |

## Énoncé

`bgp_rib.json` liste plusieurs routes candidates par préfixe. Le routeur n'en installe qu'une. Les labels des routes gagnantes, lus par préfixe croissant, forment le flag.

## Analyse

Le fichier indique lui-même le processus de décision :

1. `local_pref` le plus élevé ;
2. `as_path` le plus court ;
3. `origin` le plus bas (igp < egp < incomplete) ;
4. `med` le plus bas ;
5. `peer_id` le plus bas.

Ces critères se traduisent directement en clé de tri. On compte 45 préfixes et 137 routes, sans égalité après les cinq critères.

Deux pièges :

- **La priorité.** Les routes à `local_pref` 300 ont volontairement de longs AS paths et une origine `incomplete`, mais elles gagnent quand même. Pour `198.51.100.0/30`, un chemin de 5 sauts l'emporte sur un chemin d'un saut en `igp`, de préférence 200.
- **L'ordre des préfixes.** Il doit être numérique. Un tri de chaînes placerait `.100` avant `.12`.

## Résolution

```python
import ipaddress, json

rib = json.load(open("bgp_rib.json"))["rib"]
ORIGIN = {"igp": 0, "egp": 1, "incomplete": 2}
cle = lambda r: (-r["local_pref"], len(r["as_path"]), ORIGIN[r["origin"]], r["med"], r["peer_id"])

gagnants = []
for e in rib:
    net = int(ipaddress.ip_network(e["prefix"]).network_address)
    gagnants.append((net, min(e["routes"], key=cle)["label"]))

print("".join(label for _, label in sorted(gagnants)))
```

## Flag

```
NCTF{bgp_bestpath_localpref_beats_aspath_len}
```

## À retenir

Le `local_pref` passe avant la longueur de l'AS path. C'est ainsi qu'un opérateur impose sa politique de routage, même sur un chemin plus long.
