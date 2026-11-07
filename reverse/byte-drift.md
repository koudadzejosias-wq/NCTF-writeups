# byte-drift

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Reverse | 100 | hibris, Clawd | reverse, x86-64, beginner |

## Énoncé

Un petit programme `chall` (ELF x86-64 strippé) demande une passphrase et répond `unlocked` ou `locked`. La bonne passphrase est le flag.

## Analyse

Sans binutils sur la machine, le désassemblage de `.text` a été fait avec Capstone. La vérification tient en une boucle à `0x118e` :

1. `strlen(input) == 0x1e` : la passphrase fait 30 octets.
2. Pour chaque indice `i`, le programme calcule `rol8(input[i] + 3*i + 7, (i % 7) + 1) ^ 0x5c`.
3. Le résultat est comparé à une table de 30 octets à `0x2070` dans `.rodata`.

```asm
movzx edi, byte [rbx+rsi]   ; c = input[i]
lea   eax, [rsi+rsi*2]      ; 3*i
lea   edi, [rdi+rax+7]      ; c + 3*i + 7
...
rol   dil, cl               ; rotation de (i%7)+1
xor   edi, 0x5c
cmp   dil, byte [r9+rsi]    ; r9 = 0x2070
```

Le `imul` par `0x4924924924924925` n'est que la division par 7 optimisée par le compilateur, utilisée pour calculer `i % 7`.

Chaque étape est réversible :

```
input[i] = (ror8(enc[i] ^ 0x5c, (i%7)+1) - 3*i - 7) & 0xff
```

## Résolution

```python
enc = open("chall", "rb").read()[0x2070:0x2070 + 30]
rol = lambda b, n: ((b << n) | (b >> (8 - n))) & 0xff
ror = lambda b, n: ((b >> n) | (b << (8 - n))) & 0xff

flag = bytes((ror(enc[i] ^ 0x5c, (i % 7) + 1) - 3 * i - 7) & 0xff for i in range(30))
assert bytes(rol((flag[i] + 3 * i + 7) & 0xff, (i % 7) + 1) ^ 0x5c for i in range(30)) == enc
print(flag.decode())
```

La passphrase a été validée statiquement : le ré-encodage redonne exactement les 30 octets de la table. Le binaire n'a pas été exécuté.

## Flag

```
NCTF{inv3rt_th3_drift_by_h4nd}
```

## À retenir

Une vérification octet par octet faite d'opérations réversibles se résout par inversion directe, sans brute force.
