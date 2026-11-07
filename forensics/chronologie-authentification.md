# chronologie d'authentification

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Forensics | 150 | hibris, Clawd | forensics, analyse de logs, corrélation, facile |

## Énoncé

Un serveur public a été compromis pendant la nuit, et seul son `auth.log` a été récupéré. Il faut reconstituer les faits et retrouver la commande de l'intrus.

## Analyse

Le journal contient surtout du bruit internet : des échecs isolés, chacun venant d'une IP différente. Une IP sort du lot :

| Heure | Événement |
|---|---|
| 10:28 à 10:43 | 28 échecs SSH sur `root` et `admin` depuis `203.0.113.45` |
| 10:44:42 | `Accepted password for root` depuis cette IP, PID `25605` |
| 10:45:22 | Session root ouverte pour ce PID |
| 10:46:10 | `audit[25605]` : commande exécutée |

Le **PID** sert de clé de corrélation entre `sshd` et `audit`. La commande de l'intrus est :

```
echo ZmxhZz1OQ1RGe2JydXQzZjByYzNfdGgzbl9zdWNjM3NzXzBuX3IwMHR9 | base64 -d > /root/.ok
```

Il faut se méfier des trois sessions par clé publique vues plus tôt (`deploy`, `ubuntu`, `git`) : leurs commandes sont aussi en base64, mais elles décodent vers de l'exploitation normale (`systemctl restart nginx`, `tail -n 200 /var/log/syslog`, `git pull --ff-only origin main`). Ces sessions n'ont aucune phase de brute force.

## Résolution

```python
import base64, collections, re

lignes = open("auth.log").read().splitlines()
echecs = collections.Counter(re.search(r"from (\S+)", l).group(1) for l in lignes if "Failed password" in l)
ip, n = echecs.most_common(1)[0]
print(ip, n, "échecs")

for l in lignes:
    if f"Accepted password for root from {ip}" in l:
        pid = re.search(r"sshd\[(\d+)\]", l).group(1)
for l in lignes:
    if f"audit[{pid}]" in l:
        print(base64.b64decode(re.search(r"echo (\S+)", l).group(1)).decode())
```

La chaîne décodée est `flag=NCTF{...}`, écrite par l'intrus dans `/root/.ok`.

## Flag

```
NCTF{brut3f0rc3_th3n_succ3ss_0n_r00t}
```

## À retenir

La séquence « série d'échecs depuis une IP, puis succès depuis la même IP » est la signature du brute force réussi. Le PID relie ensuite la connexion aux actions effectuées.
