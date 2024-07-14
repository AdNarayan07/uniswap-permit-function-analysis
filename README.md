# Uniswap `permit` function analysis
## Introduction
*  Protocol Name: Uniswap
*  Category: DeFi
*  Smart Contract: Uni

## Function Analysis
- Function Name: `permit`
- Block Explorer Link: https://etherscan.io/token/0x1f9840a85d5af5bf1d1762f925bdaddc4201f984

- Function Code:
```solidity
function permit(address owner, address spender, uint rawAmount, uint deadline, uint8 v, bytes32 r, bytes32 s) external {
    uint96 amount;
    if (rawAmount == uint(-1)) {
        amount = uint96(-1);
    } else {
        amount = safe96(rawAmount, "Uni::permit: amount exceeds 96 bits");
    }

    bytes32 domainSeparator = keccak256(abi.encode(DOMAIN_TYPEHASH, keccak256(bytes(name)), getChainId(), address(this)));
    bytes32 structHash = keccak256(abi.encode(PERMIT_TYPEHASH, owner, spender, rawAmount, nonces[owner]++, deadline));
    bytes32 digest = keccak256(abi.encodePacked("\x19\x01", domainSeparator, structHash));
    address signatory = ecrecover(digest, v, r, s);
    require(signatory != address(0), "Uni::permit: invalid signature");
    require(signatory == owner, "Uni::permit: unauthorized");
    require(now <= deadline, "Uni::permit: signature expired");

    allowances[owner][spender] = amount;

    emit Approval(owner, spender, amount);
}
```
- Used Encoding/Decoding or Call Method: `abi.encode` and `abi.encodePacked`

## Explanation

- Purpose:
The permit function belongs to the UNI governance token and allows an off-chain signature to enable the spending of tokens on behalf of the owner. This function also follows the ERC-2612 standard created by the community, which intends to build upon the ERC-20 token standard by including approvals through signatures.

- Detailed Usage:
    * `abi.encode`: This function plays a key role in the permit function to generate data hashes:

        * `abi.encode(DOMAIN_TYPEHASH keccak256(bytes(name)) getChainId(), address(this))`: This creates the domain separator for EIP-712 making sure the signature applies to this contract and the current chain.
        * `abi.encode(PERMIT_TYPEHASH owner, spender, rawAmount, nonces[owner]++ deadline)`: This creates the struct hash for the permit, which includes the owner, spender, amount, nonce, and deadline.
        
        The `abi.encode` function joins different data pieces into one padded byte array. To make sure domain and struct hashes stay the same and unique, it first turns all inputs into a 32-byte boundary alignment.

    * `abi.encodePacked`: This combines the domain separator and struct hash into one digest for signing:
        * `abi.encodePacked("\x19\x01" domainSeparator, structHash)`: This packed encoding starts with the standard header ("\x19\x01") for EIP-712 typed data then adds the encoded domain separator and struct hash. It uses the packed format here to make a unique message digest for the token owner to sign.
        
        This function helps concatenate multiple data pieces into a single byte array without padding. This results in a more compact representation that is very useful in creating the final message digest to be signed. The compactness of the representation makes the size of the data being hashed smaller and hence efficient.

## Impact:
Using the permit function, token transfers can be approved without initiating an on-chain transaction, a process that saves users on gas costs as well as enhancing their experience.As for ab1.encode and abi.encodePacked, they certify that when data is signed; it will be specific to just this contract alone, its chain while ensuring that no replay attacks happen across other networks. This is an important aspect in making secure and implementable permits; hence enhancing security and functional aspects of it.
