import hashlib
import json
from datetime import datetime
from dataclasses import dataclass, field
from typing import Any, Dict, List

def sha256(text: str) -> str:
    return hashlib.sha256(text.encode("utf-8")).hexdigest()

@dataclass
class Block:
    index: int
    timestamp: str
    data: Dict[str, Any]
    previous_hash: str = "0"
    nonce: int = 0
    hash: str = field(init=False)

    def __post_init__(self):
        self.hash = self.compute_hash()

    def compute_hash(self) -> str:
        payload = f"{self.index}{self.timestamp}{json.dumps(self.data, sort_keys=True)}{self.previous_hash}{self.nonce}"
        return sha256(payload)

class Blockchain:
    def __init__(self, difficulty: int = 3):
        self.chain: List[Block] = [self._create_genesis_block()]
        self.difficulty = difficulty

    def _create_genesis_block(self) -> Block:
        return Block(
            index=0,
            timestamp=datetime.utcnow().isoformat(),
            data={"genesis": True},
            previous_hash="0",
        )

    def latest_block(self) -> Block:
        return self.chain[-1]

    def mineBlock(self, block: Block) -> Block:
        target = "0" * self.difficulty
        while not block.hash.startswith(target):
            block.nonce += 1
            block.hash = block.compute_hash()
        return block

    def setBlock(self, data: Dict[str, Any]) -> Block:
        idx = len(self.chain)
        prev_hash = self.latest_block().hash
        new_block = Block(
            index=idx,
            timestamp=datetime.utcnow().isoformat(),
            data=data,
            previous_hash=prev_hash,
        )
        self.mineBlock(new_block)
        self.chain.append(new_block)
        return new_block

    def blocksExplorer(self) -> List[Dict[str, Any]]:
        return [
            {
                "index": b.index,
                "timestamp": b.timestamp,
                "data": b.data,
                "nonce": b.nonce,
                "prevHash": b.previous_hash[:16] + "...",
                "hash": b.hash[:16] + "...",
            }
            for b in self.chain
        ]

    def is_valid(self) -> bool:
        if not self.chain:
            return True
        target = "0" * self.difficulty
        for i in range(1, len(self.chain)):
            prev = self.chain[i - 1]
            cur = self.chain[i]
            if cur.previous_hash != prev.hash:
                return False
            if cur.hash != cur.compute_hash():
                return False
            if not cur.hash.startswith(target):
                return False
        return True

if __name__ == "__main__":
    bc = Blockchain(difficulty=3)
    bc.setBlock({"from": "Ali", "to": "Mazen", "amount": 25})
    bc.setBlock({"note": "Second block"})
    for row in bc.blocksExplorer():
        print(row)
