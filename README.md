# Writeup NCTF

Writeups des challenges NCTF résolus sur la plateforme [arena.ctf.tg](https://arena.ctf.tg).

**39 challenges, 6970 points.**

Chaque writeup suit le même plan : énoncé résumé, analyse, script de résolution, flag et point à retenir.

## Reproduire une résolution

Les fichiers des challenges ne sont pas inclus dans ce dépôt. Pour rejouer un script, placez-le dans le dossier du challenge. Quelques scripts demandent `pillow`, `numpy` ou `pycryptodome`.

Le code de chaque writeup a été réexécuté sur les fichiers d'origine et redonne le flag indiqué. Il y a deux exceptions, dont le flag se lit dans une image : `flush` et `mirage-1`. Ces images ont été régénérées et relues.

Le script d'`intel-hex` embarque directement les lignes utiles du dump, car le fichier d'origine n'était plus disponible.

Le format du flag de [`djobokou`](crypto/djobokou.md) n'a pas été confirmé : voir la note dans le writeup.

## Crypto

| Challenge | Points | Auteur |
|---|---:|---|
| [sodabi](crypto/sodabi.md) | 98 | KodjoDoDjango |
| [rsa-beginer](crypto/rsa-beginer.md) | 100 | thierry8petstar |
| [djobokou](crypto/djobokou.md) | 246 | KodjoDoDjango |
| [bassar](crypto/bassar.md) | 249 | KodjoDoDjango |
| [niamtougou](crypto/niamtougou.md) | 249 | KodjoDoDjango |
| [koklo](crypto/koklo.md) | 250 | KodjoDoDjango |
| [kpalime](crypto/kpalime.md) | 400 | KodjoDoDjango |
| [sakpata](crypto/sakpata.md) | 400 | KodjoDoDjango |

## Reverse

| Challenge | Points | Auteur |
|---|---:|---|
| [shell-lock](reverse/shell-lock.md) | 86 | hibris, Clawd |
| [byte-drift](reverse/byte-drift.md) | 100 | hibris, Clawd |
| [strings-lie](reverse/strings-lie.md) | 150 | hibris, Clawd |
| [triple-wrap](reverse/triple-wrap.md) | 150 | hibris, Clawd |

## IA

| Challenge | Points | Auteur |
|---|---:|---|
| [oracle-doc-injection](ai/oracle-doc-injection.md) | 146 | hibris, Clawd |
| [member-ids](ai/member-ids.md) | 150 | hibris, Clawd |
| [oracle-fake-system](ai/oracle-fake-system.md) | 150 | hibris, Clawd |
| [oracle-reverse-emit](ai/oracle-reverse-emit.md) | 150 | hibris, Clawd |
| [oracle-canary-decoy](ai/oracle-canary-decoy.md) | 298 | hibris, Clawd |
| [oracle-multi-turn](ai/oracle-multi-turn.md) | 394 | hibris, Clawd |

## Supply chain

| Challenge | Points | Auteur |
|---|---:|---|
| [sbom-component-swap](supply-chain/sbom-component-swap.md) | 111 | hibris, Clawd |
| [build-cache-poison](supply-chain/build-cache-poison.md) | 150 | hibris, Clawd |
| [typosquat-lockfile](supply-chain/typosquat-lockfile.md) | 150 | hibris, Clawd |
| [dependency-confusion](supply-chain/dependency-confusion.md) | 300 | hibris, Clawd |

## Forensics

| Challenge | Points | Auteur |
|---|---:|---|
| [png-magic-fix](forensics/png-magic-fix.md) | 97 | hibris, Clawd |
| [mirage-1](forensics/mirage-1.md) | 100 | The-darknova |
| [chronologie d'authentification](forensics/chronologie-authentification.md) | 150 | hibris, Clawd |
| [flush](forensics/flush.md) | 200 | Oxdefault |
| [trail-of-keys](forensics/trail-of-keys.md) | 300 | hibris, Clawd |

## Stéganographie

| Challenge | Points | Auteur |
|---|---:|---|
| [lsb-bmp](stego/lsb-bmp.md) | 89 | hibris, Clawd |
| [trailing-zip](stego/trailing-zip.md) | 100 | hibris, Clawd |
| [png-text-chunk](stego/png-text-chunk.md) | 150 | hibris, Clawd |
| [greffe-espaces-lome](stego/greffe-espaces-lome.md) | 175 | Algorithme123 |
| [tisserands-dapaong](stego/tisserands-dapaong.md) | 250 | Algorithme123 |

## Réseau

| Challenge | Points | Auteur |
|---|---:|---|
| [eui64-slaac](network/eui64-slaac.md) | 100 | hibris, Clawd |
| [bgp-bestpath](network/bgp-bestpath.md) | 150 | hibris, Clawd |

## Blockchain

| Challenge | Points | Auteur |
|---|---:|---|
| [block-oracle](blockchain/block-oracle.md) | 132 | hibris, Clawd |

## Hardware

| Challenge | Points | Auteur |
|---|---:|---|
| [intel-hex](hardware/intel-hex.md) | 100 | hibris, Clawd |

## Misc

| Challenge | Points | Auteur |
|---|---:|---|
| [kabou](misc/kabou.md) | 100 | KodjoDoDjango |

## PPC

| Challenge | Points | Auteur |
|---|---:|---|
| [hull-cipher](ppc/hull-cipher.md) | 150 | hibris, Clawd |

## Sysadmin

| Challenge | Points | Auteur |
|---|---:|---|
| [alias-slip](sysadmin/alias-slip.md) | 150 | hibris, Clawd |

## Droits d'auteur

© 2026 KOUDADZE Kodjogan Josias. Tous droits réservés.

Ces writeups sont l'œuvre de leur auteur. Toute reproduction ou republication, totale ou partielle, sans son autorisation écrite est interdite. Les challenges eux-mêmes appartiennent à leurs auteurs respectifs et à la plateforme NCTF.
