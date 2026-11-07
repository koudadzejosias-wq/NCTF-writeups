# flush

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Forensics | 200 | Oxdefault | forensics, medium |

## Énoncé

Un sous-traitant sur le départ a exfiltré des documents par USB, pendant qu'une capture `usbmon` tournait. Premier épisode : retrouver le fichier copié sur une clé de stockage de masse, le flag est dedans.

## Analyse

`capture.pcap` est une capture usbmon (linktype 220, en-tête de 64 octets par trame) de 30 paquets, sur le périphérique 7. Les endpoints bulk `0x02` et `0x81` indiquent du **USB Mass Storage** (Bulk-Only Transport).

Chaque commande est un CBW commençant par `USBC`, qui contient une commande SCSI :

```
INQUIRY     (0x12)
READ CAPACITY (0x25)
READ(10)  LBA=0     1 bloc    secteur de boot FAT
WRITE(10) LBA=2051  1 bloc
WRITE(10) LBA=2063  6 blocs
WRITE(10) LBA=2069  3 blocs
WRITE(10) LBA=2057  6 blocs
WRITE(10) LBA=2052  5 blocs
WRITE(10) LBA=2072  2 blocs
WRITE(10) LBA=2048  3 blocs
```

Le piège est là : les écritures **ne sont pas dans l'ordre**. Concaténer les paquets donnerait un fichier mélangé. Il faut replacer chaque bloc à son LBA, comme le ferait le disque.

Une fois l'image reconstruite, les données à partir du LBA 2048 forment un PNG complet de 760×220. Il n'a aucun chunk de texte : le flag est écrit dans l'image.

## Résolution

```python
import struct

d = open("capture.pcap", "rb").read()
off, recs = 24, []
while off + 16 <= len(d):
    _, _, cl, _ = struct.unpack("<IIII", d[off:off + 16])
    off += 16
    p = d[off:off + cl]
    off += cl
    if len(p) >= 64:
        recs.append(p[64:64 + struct.unpack("<I", p[36:40])[0]])

img, cur = {}, None
for data in recs:
    if data[:4] == b"USBC":
        cb = data[15:15 + data[14]]
        cur = None
        if cb[0] in (0x28, 0x2a):
            cur = struct.unpack(">I", cb[2:6])[0]
    elif data[:4] == b"USBS":
        cur = None
    elif cur is not None:
        for i in range(0, len(data), 512):
            img[cur + i // 512] = data[i:i + 512].ljust(512, b"\0")
        cur = None

disque = b"".join(img.get(i, b"\0" * 512) for i in range(max(img) + 1))
s = disque[2048 * 512:]
png = s[s.find(b"\x89PNG"):s.find(b"IEND") + 8]
open("flag.png", "wb").write(png)
print(len(png), "octets, voir flag.png")
```

Le flag se lit ensuite directement dans `flag.png`.

## Flag

```
NCTF{t0ut_c3_qu1_3st_3cr1t_r3st3}
```

## À retenir

Sur une capture de stockage de masse, l'ordre des paquets n'est pas l'ordre du fichier. C'est le LBA qui compte.
