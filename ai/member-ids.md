# member-ids

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| IA / ML | 150 | hibris, Clawd | ml, membership-inference, privacy, easy |

## Énoncé

`shadow_eval.npz` donne, pour chaque enregistrement, un identifiant, la perte du modèle et un tag d'un caractère. Les tags des enregistrements du jeu d'entraînement, lus dans le bon ordre, forment le flag.

## Analyse

C'est une **inférence d'appartenance**. Un modèle surappris a une perte bien plus faible sur les données qu'il a vues. Les 80 pertes triées se séparent nettement en deux groupes :

```
0,0114 ... 0,1360   36 enregistrements (membres)
   ------ écart de 0,136 à 0,560 ------
0,5603 ... 1,1872   44 enregistrements (hors entraînement)
```

Tout seuil placé dans cet écart donne les mêmes 36 membres. Trier ces membres par `record_id` et convertir les tags (des codes ASCII) donne 36 caractères. Les 44 autres tags ne donnent que du bruit, ce qui confirme la séparation.

## Résolution

```python
import numpy as np

d = np.load("shadow_eval.npz")
rid, loss, tag = d["record_id"], d["loss"], d["tag"]

membres = loss < 0.3
ordre = np.argsort(rid[membres])
print("".join(chr(c) for c in tag[membres][ordre]))
```

## Flag

```
NCTF{loss_gap_outs_the_training_set}
```

## À retenir

Un écart de perte entre entraînement et test est une fuite de confidentialité. La régularisation et la confidentialité différentielle réduisent cet écart.
