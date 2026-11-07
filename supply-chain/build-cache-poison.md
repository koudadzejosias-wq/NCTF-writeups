# build-cache-poison

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Supply chain | 150 | hibris, Clawd | supplychain, build-cache, makefile, easy |

## Énoncé

On reçoit le Makefile qui alimente un cache de build, sa liste `checksums.sha256` et le contenu de `vendor-cache/`. Une dépendance n'est pas ce qu'elle prétend être.

## Analyse

Le Makefile désigne le suspect :

```make
vendor-cache/libnet-0.9.tar.gz:
	curl -fsSL -o $@ http://mirror.build-cache.dev/libnet-0.9.tar.gz
```

| Artefact | Source | Vérification | Dans `checksums.sha256` |
|---|---|---|---|
| `libcrc-1.0` | `https://cdn.acme.dev` | oui | oui |
| `libui-2.3` | `https://cdn.acme.dev` | oui | oui |
| `libz-1.2` | `https://cdn.acme.dev` | oui | oui |
| **`libnet-0.9`** | `http://mirror.build-cache.dev` | **non** | **non** |

`libnet` vient d'un miroir tiers en HTTP clair, sans contrôle. Même les autres cibles ne le détecteraient pas : `sha256sum --ignore-missing -c` ignore silencieusement les fichiers absents du manifeste.

Les trois archives saines contiennent un `src.c` d'une ligne. `libnet` ne contient aucune source, seulement un `build.env` :

```
BUILD_TOKEN=cn9oekdeSVVQWGNfXV9UWWNSU2NfVFlfV09JUWNMSVBQYwRYD1pB
```

Le jeton est en base64, puis XOR avec la clé du Makefile, `XOR_KEY := 60`. Cette valeur est décimale : c'est `0x3C`, pas `0x60`.

## Résolution

```python
import base64, re, tarfile

with tarfile.open("libnet-0.9.tar.gz") as t:
    env = t.extractfile("build.env").read().decode()

jeton = re.search(r"BUILD_TOKEN=(\S+)", env).group(1)
print(bytes(c ^ 60 for c in base64.b64decode(jeton)).decode())
```

## Flag

```
NCTF{build_cache_no_checksum_pull_8d3f}
```

## À retenir

Une dépendance non épinglée échappe à toute la chaîne de vérification. `--ignore-missing` rend ce trou invisible : mieux vaut que tout fichier non listé fasse échouer le build.
