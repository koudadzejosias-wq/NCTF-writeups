# mirage-1

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Forensics | 100 | The-darknova | forensics, easy |

## Énoncé

Le SOC a repéré une transmission inhabituelle sur le réseau, et l'analyste de garde dit avoir vu passer un « fantôme » dans le flux.

## Analyse

Les statistiques de `mirage-1.pcapng` (7 550 trames) font ressortir 3 758 datagrammes UDP de 1 336 octets vers le port 5000, soit l'essentiel du volume. Le début des paquets, `80 21 ...`, désigne de l'**RTP** de type 33, c'est-à-dire du MPEG-TS.

La reconstruction se fait en trois étapes :

1. extraire les charges UDP avec `tshark` ;
2. retirer les 12 octets d'en-tête RTP et trier par numéro de séquence (aucune perte sur les 3 758 paquets) ;
3. concaténer le tout en flux `.ts`.

`ffprobe` y trouve 29,65 s de vidéo H.264 en 360×640 et de l'audio AAC. Le flag est incrusté en surimpression sur la vidéo.

## Résolution

```bash
tshark -r mirage-1.pcapng -Y "udp.dstport==5000" -T fields -e data.data > payloads.txt
```

```python
paquets = {}
for ligne in open("payloads.txt"):
    b = bytes.fromhex(ligne.strip())
    paquets[int.from_bytes(b[2:4], "big")] = b[12:]

seq = sorted(paquets)
print(len(seq), "paquets, pertes :", seq[-1] - seq[0] + 1 - len(seq))
open("stream.ts", "wb").write(b"".join(paquets[i] for i in seq))
```

```bash
ffmpeg -i stream.ts -vf "select=eq(n\,299)" -vsync 0 -frames:v 1 frame.png
```

Le flag se lit en haut de l'image extraite. Un agrandissement de la zone facilite la lecture.

## Flag

```
NCTF{oohhh_a_ghost_gg}
```

> Le flag a été lu à l'œil sur l'image.

## À retenir

Un gros flux UDP régulier sur un port arbitraire est souvent du média. Il faut alors reconstruire le flux et le regarder, car un `strings` n'y trouvera rien.
