# sbom-component-swap

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Supply chain | 111 | hibris, Clawd | supplychain, sbom, cyclonedx, easy |

## Énoncé

On reçoit le SBOM CycloneDX d'une release (`bom.json`), les binaires des composants et un flux d'empreintes malveillantes du SOC (`known-bad-hashes.txt`). Il faut identifier le composant compromis.

## Analyse

Le pivot consiste à hacher chaque binaire, puis à croiser le résultat avec le flux du SOC :

| Composant | SHA-256 | Conforme au SBOM | Dans le flux |
|---|---|---|---|
| libjpeg-turbo 3.0.1 | `aa913f31…9052` | oui | non |
| zlib 1.3 | `bda0ebc9…5382` | oui | non |
| **acme-updater 2.4.0** | **`cb512bc9…892b`** | oui | **`acme-updater-backdoor`** |
| openssl 3.2.1 | `3b5bc334…7fc4` | oui | non |

Point important : **les quatre binaires correspondent à leur empreinte déclarée**, donc une vérification du SBOM seule ne signale rien. Les autres entrées du flux sont des leurres : `e3b0c442…` est le SHA-256 de la chaîne vide, et les deux autres ceux de `"0"` et `"1"`.

Le binaire `acme-updater` contient une variable `cfg` en base64.

## Résolution

```python
import base64, glob, hashlib, re

mauvais = {}
for l in open("known-bad-hashes.txt"):
    if l.strip() and not l.startswith("#"):
        h, nom = l.split()
        mauvais[h] = nom

for f in sorted(glob.glob("component-*.bin")):
    d = open(f, "rb").read()
    h = hashlib.sha256(d).hexdigest()
    if h in mauvais:
        print(f, "->", mauvais[h])
        print(base64.b64decode(re.search(rb"cfg = '([^']+)'", d).group(1)).decode())
```

## Flag

```
NCTF{sb0m_component_hash_pivot_9f4c}
```

## À retenir

Un SBOM cohérent dit ce qui a été livré, pas si c'est sain. La détection n'apparaît qu'en croisant les empreintes vérifiées avec le renseignement sur les menaces.
