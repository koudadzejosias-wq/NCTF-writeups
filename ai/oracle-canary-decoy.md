# oracle-canary-decoy

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| IA | 298 | hibris, Clawd | ai, guardrail-bypass, reversing, medium |

## Énoncé

ChannelBot donne un flag dès qu'on le demande, mais cette réponse ne rapporte rien : il sert plusieurs canaux et un seul dit la vérité.

## Analyse

1. **Le leurre.** Tout prompt contenant `flag`, `secret`, `key` ou `token` déclenche `_asks_secret()` et renvoie le canari `NCTF{th1s_1s_0nly_4_d3c0y_k33p_l00k1ng}`.
2. **Le vrai chemin.** `respond()` contrôle les en-têtes **avant** cette branche. `_headers()` traite toute ligne `clé: valeur` du prompt comme un en-tête : le corps du prompt sert donc de canal d'en-têtes.
3. **Le jeton.** `_TOK` est une liste d'octets XOR `0x5B`, qui donne `maint-override-9f2c7a`.
4. **L'appel.** `X-Auth: maint-override-9f2c7a` ouvre la branche admin, qui décode `_REAL` (base64, puis inversion).

## Résolution

```python
import importlib.util

spec = importlib.util.spec_from_file_location("oracle", "oracle.py")
m = importlib.util.module_from_spec(spec)
spec.loader.exec_module(m)

print(m.respond("X-Auth: " + m._admin_token()))
```

## Flag

```
NCTF{r34l_fl4g_l1v3s_1n_4dm1n_ch4nn3l}
```

## À retenir

Tous les flags ne se valent pas : `th1s_1s_0nly_4_d3c0y` est le leurre. Un secret d'authentification livré dans le code, même obfusqué par XOR, reste lisible.
