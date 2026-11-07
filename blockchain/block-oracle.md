# block-oracle

| Catégorie | Points | Auteur | Tags |
|---|---|---|---|
| Blockchain | 132 | hibris, Clawd | blockchain, evm, randomness, easy |

## Énoncé

Une loterie scelle sa cagnotte pour celui qui devine le ticket gagnant. On reçoit `BlockLottery.sol` et `draw.json`, qui contient le contexte exact du bloc et le chiffré `cipher_hex`.

## Analyse

Le contrat tire son « hasard » du bloc lui-même :

```solidity
return uint256(keccak256(abi.encodePacked(
    block.timestamp, block.number, block.prevrandao)));
```

Ces trois valeurs sont dans `draw.json`, et ce sont trois `uint256`. L'image de la fonction de hachage fait donc simplement 3×32 octets. Aucun accès à la chaîne n'est nécessaire.

Le ticket donne le flux de déchiffrement. La construction exacte n'étant pas dans le contrat, plusieurs variantes ont été testées avec le préfixe `NCTF` comme contrôle. La bonne est un mode compteur :

```
bloc[i] = keccak256(ticket_bytes32 || uint32be(i)),  i = 0, 1, ...
```

## Résolution

Attention : Ethereum utilise Keccak-256 original, différent de `hashlib.sha3_256`.

```python
import json
from Crypto.Hash import keccak

k = lambda b: keccak.new(digest_bits=256, data=b).digest()

d = json.load(open("draw.json"))
b = d["block"]
ct = bytes.fromhex(d["cipher_hex"])

ticket = k(b["timestamp"].to_bytes(32, "big")
           + b["number"].to_bytes(32, "big")
           + int(b["prevrandao"], 16).to_bytes(32, "big"))

flux, i = b"", 0
while len(flux) < len(ct):
    flux += k(ticket + i.to_bytes(4, "big"))
    i += 1
print(bytes(x ^ y for x, y in zip(ct, flux)).decode())
```

## Flag

```
NCTF{blockhash_is_not_randomness}
```

## À retenir

Tout ce qu'un contrat lit dans le bloc est public et, pour partie, influençable par les validateurs. Un vrai tirage nécessite une source externe, comme un VRF ou un schéma commit-reveal.
