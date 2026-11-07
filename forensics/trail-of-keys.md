# trail-of-keys

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Forensics / Cloud | 300 | hibris, Clawd | cloud, aws, cloudtrail, incident-response, medium |

## Énoncé

Un export CloudTrail d'une heure sur le compte `kekeli-prod` et un objet `exfil-notes.json` écrit par une session dans un bucket temporaire. Quelqu'un a lu un secret sensible et l'a sorti du compte.

## Analyse

Le suivi se fait par clés d'accès temporaires (`ASIA...`) :

| Heure | Identité | Action |
|---|---|---|
| 09:07:22 | `AKIALEAK0MALLORY0001` (`session-9f2c`) | `AssumeRole` sur `support-readonly`, obtient `ASIA5EXFIL0SESSION07` |
| 09:07:58 | `ASIA5EXFIL0SESSION07` | `ListSecrets` |
| 09:08:31 | `ASIA5EXFIL0SESSION07` | `GetSecretValue` sur `prod/customer-export-Kd93js` |
| 09:09:12 | `ASIA5EXFIL0SESSION07` | `PutObject` vers `kekeli-scratch/tmp/notes.bin` |

Il y a un leurre : à 09:11:20, `bob-cli` lit aussi un secret avec `ASIA5DECOY0SESSION09`, mais c'est `dev/grafana`, un secret sans rapport avec la production.

`exfil-notes.json` indique que le flux XOR vaut `SHA256(<clé ASIA>)`. Le chiffré fait 40 octets et un SHA-256 seulement 32. Il faut donc comprendre comment le flux est étendu.

Une répétition du condensat est exclue : le XOR des octets 0 à 7 et 32 à 39 a des bits de poids fort à 1, ce qui est impossible pour deux textes ASCII. Une recherche sur les constructions courantes, avec le préfixe connu `NCTF{` comme test, donne la bonne :

```
bloc[i] = SHA256( SHA256(clé) || uint32be(i) ),  i = 0, 1, ...
```

## Résolution

```python
import base64, hashlib, json

ct = base64.b64decode(json.load(open("exfil-notes.json"))["ciphertext_b64"])
d = hashlib.sha256(b"ASIA5EXFIL0SESSION07").digest()

flux, i = b"", 0
while len(flux) < len(ct):
    flux += hashlib.sha256(d + i.to_bytes(4, "big")).digest()
    i += 1
print(bytes(a ^ b for a, b in zip(ct, flux)).decode())
```

## Flag

```
NCTF{cloudtrail_traced_the_asia_session}
```

## À retenir

Dans CloudTrail, `responseElements.credentials.accessKeyId` relie un `AssumeRole` à toutes les actions de la session. C'est ce lien qui remonte d'une clé `ASIA` temporaire jusqu'à la clé `AKIA` compromise.
