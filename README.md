# MerkleTree.js Solidity example

> Construct merkle trees with [MerkleTree.js](https://github.com/miguelmota/merkletreejs) and verify merkle proofs in [Solidity](https://github.com/ethereum/solidity).

## Example

[`contracts/MerkleProof.sol`](./contracts/MerkleProof.sol)

```solidity
pragma solidity ^0.5.2;

contract MerkleProof {
  function verify(
    bytes32 root,
    bytes32 leaf,
    bytes32[] memory proof
  )
    public
    pure
    returns (bool)
  {
    bytes32 computedHash = leaf;

    for (uint256 i = 0; i < proof.length; i++) {
      bytes32 proofElement = proof[i];

      if (computedHash < proofElement) {
        // Hash(current computed hash + current element of the proof)
        computedHash = keccak256(abi.encodePacked(computedHash, proofElement));
      } else {
        // Hash(current element of the proof + current computed hash)
        computedHash = keccak256(abi.encodePacked(proofElement, computedHash));
      }
    }

    // Check if the computed hash (root) is equal to the provided root
    return computedHash == root;
  }
}
```

[`test/merkleproof.js`](./test/merkleproof.js)

```js
const MerkleProof = artifacts.require('MerkleProof')
const MerkleTree = require('merkletreejs')
const keccak256 = require('keccak256')

const buf2hex = x => '0x'+x.toString('hex')
const contract = await MerkleProof.new()

const leaves = ['a', 'b', 'c', 'd']
const tree = new MerkleTree(leaves, keccak256, {
  hashLeaves: true,
  sortLeaves: true,
  sortPairs: true,
})
const root = tree.getHexRoot()
const leaf = tree.leaves[0]
const proof = tree.getHexProof(leaf)

const verified = await contract.verify.call(root, leaf, proof)

console.log(verified) // true
```

## Test

```bash
make test
```

## License

MIT
