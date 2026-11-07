# strings-lie

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Reverse | 150 | hibris, Clawd | reverse, beginner |

## Énoncé

Un binaire « coffre secret » affiche un flag dans sa bannière, mais ce flag est refusé. Il faut retrouver le vrai.

## Analyse

Le `.rodata` contient le leurre `NCTF{y0u_grepp3d_th3_f4ke_0ne}` et deux blobs illisibles :

| Adresse | Taille | Rôle |
|---|---:|---|
| `0x20a0` | 31 | flag chiffré |
| `0x20c0` | 23 | passphrase encodée |

**Vérification de la passphrase** (`0x11a1`), après `strlen(input) == 0x17` :

```asm
movzx eax, byte [rbx+rdx]   ; input[i]
xor   eax, 0x5a
add   eax, ecx              ; ecx démarre à 3, +7 à chaque tour
cmp   al, byte [rsi+rdx]    ; blobB[i]
```

Soit `((input[i] ^ 0x5a) + 3 + 7*i) & 0xff == blobB[i]`, qui s'inverse immédiatement.

**Déchiffrement du flag** (`0x1268`). La séquence `movabs 0x642c8590b21642c9` / `mul` / `shr` est une division par 23 optimisée par le compilateur, qui calcule `i % 23` :

```
flag[i] = blobA[i] ^ passphrase[i % 23]
```

Détail utile : le blob A est référencé par `lea r8` (`4c 8d 05`). Un premier balayage limité aux formes `48 8d` ne le trouvait pas.

Autre piste de contrôle : `blobA ⊕ "NCTF{"` donne `unw1n`, le début de la passphrase.

## Résolution

```python
b = open("chall", "rb").read()
A = b[0x20a0:0x20a0 + 31]
B = b[0x20c0:0x20c0 + 23]

pw = bytes(((B[i] - (3 + 7 * i)) & 0xff) ^ 0x5a for i in range(23))
print("passphrase :", pw.decode())
print(bytes(A[i] ^ pw[i % 23] for i in range(31)).decode())
```

La passphrase est `unw1nd_th3_math_by_h4nd`.

## Flag

```
NCTF{str1ngs_l1e_read_the_c0de}
```

## À retenir

`strings` montre ce que l'auteur veut bien montrer. C'est le code qui fait foi.
