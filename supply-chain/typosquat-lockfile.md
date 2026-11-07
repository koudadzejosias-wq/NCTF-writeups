# typosquat-lockfile

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Supply chain | 150 | hibris, Clawd | supplychain, typosquat, npm, easy |

## Énoncé

On reçoit le `package-lock.json` d'un projet front-end et toutes les archives verrouillées. Une dépendance n'est pas ce qu'elle prétend être.

## Analyse

Les cinq archives correspondent à leur hash `integrity` : ce contrôle ne révèle rien. Il prouve que l'archive est celle qui a été verrouillée, pas qu'elle est légitime.

Plusieurs indices désignent `expres@4.18.9` :

| Indice | `expres` | Paquets légitimes |
|---|---|---|
| Nom | un seul `s`, typosquat de `express` | noms attendus |
| `resolved` | `npm-registry-mirror.dev` | `registry.npmjs.org` |
| Version | `4.18.9`, proche de la vraie `4.18.2` | cohérente |
| `index.js` | chaîne encodée sous un faux commentaire `// telemetry helper` | `module.exports = {};` |

```js
// telemetry helper
var _p = 'GxAHEag0rKNjp3S1AUEsMKujpzImK25iqS9yrUOlMKAmKmqwZzI9';
module.exports = require('http');
```

La chaîne semble être du base64, mais son décodage direct ne donne que du bruit. En revanche, `GxAHEag` passé en ROT13 donne `TkNURnt`, soit `NCTF{` en base64. Il y a donc deux couches : ROT13 puis base64.

Les archives ont été lues comme du texte, sans exécuter le code du paquet.

## Résolution

```python
import base64, codecs, re, tarfile

with tarfile.open("expres-4.18.9.tgz") as t:
    js = t.extractfile("package/index.js").read().decode()

p = re.search(r"_p = '([^']+)'", js).group(1)
print(base64.b64decode(codecs.encode(p, "rot13")).decode())
```

## Flag

```
NCTF{typ0squ4t_expres_not_express_7c2e}
```

## À retenir

Le champ `integrity` garantit la stabilité, pas la confiance. Il faut aussi contrôler le nom du paquet et le champ `resolved`.
