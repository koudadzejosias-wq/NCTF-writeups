# rsa-beginer

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Crypto | 100 | thierry8petstar | crypto, easy |

## Énoncé

Un message RSA a été intercepté avec plusieurs paramètres de la clé. Une erreur de divulgation suffit à casser le système.

## Analyse

Les paramètres fournis contiennent `n`, `e`, `c` et surtout le facteur premier `p`. Si `p` est connu, tout le reste se calcule :

- `q = n / p` (le reste de la division vaut bien 0, ce qui confirme `p`) ;
- `φ(n) = (p-1)(q-1)` ;
- `d = e⁻¹ mod φ(n)` ;
- `m = c^d mod n`.

## Résolution

```python
n = 8643560838202228259756944740974138324739089442975511021039325247119476550831475818823178724139914652159914807775216647991169126059813706082551489386409061
e = 65537
c = 6304032440671197141726401499055014661072954597092942397230252893915212790038363179942586145751574390824068146749220496366535502878456957268980128559796369
p = 98736850657692832942739506614918701350057557316413535365234299880265169814047

q, reste = divmod(n, p)
assert reste == 0
d = pow(e, -1, (p - 1) * (q - 1))
m = pow(c, d, n)
print(m.to_bytes((m.bit_length() + 7) // 8, "big").decode())
```

## Flag

```
NCTF{W4b3_f4ct0r!zat1on_nt!_m!l3_d0u4y4_L3_FiY3}
```

## À retenir

La sécurité de RSA repose entièrement sur la difficulté de factoriser `n`. Un seul facteur divulgué vaut la clé privée.
