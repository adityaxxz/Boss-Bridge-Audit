# Boss Bridge Audit

- [Audit-Report](#audit-report)
- [What's Boss Bridge](#whats-boss-bridge)
- [Boss Bridge Explained in a nutshell](#boss-bridge-explained-in-a-nutshell)
  - [Main Contract](#main-contract)
  - [How Tokens Travel Between Layers](#how-tokens-travel-between-layers)
  - [The Key Role of Signers](#the-key-role-of-signers)

# Audit Report 

[Audit-Report-HERE](audit-data/report.pdf)


## What's Boss Bridge

This project presents a simple bridge mechanism to move our ERC20 token from L1 to an L2.

In a nutshell, the bridge allows users to deposit tokens, which are held into a secure vault on L1. Successful deposits trigger an event that our off-chain mechanism picks up, parses it and mints the corresponding tokens on L2.

To ensure user safety, this first version of the bridge has a few security mechanisms in place:

- The owner of the bridge can pause operations in emergency situations.
- Because deposits are permissionless, there's an strict limit of tokens that can be deposited.
- Withdrawals must be approved by a bridge operator.

## Boss Bridge Explained in a nutshell

<p align="center">
<img src = "audit-data/diagrams/Boss-Bridge.png" width=850>
<br/>

### <u>Main Contract</u>

The `L1BossBridge.sol` contract has a substantial role and a few capabilities. It can pause and unpause, illustrating some centralized power. Most crucially, it permits users to deposit tokens to L2 and withdraw tokens from the L2 back to the L1.

```javascript
function sendToL2(address _l2Delegate, address _token, uint256 _amount, uint256 _l2Gas, bytes calldata _data) external whenNotPaused returns (bytes memory){
    // (...rest of code...)
}
```

The `sendToL2()` function deposits token to L2. Once tokens are sent, they are locked into `L1Vault.sol`. This vault is relatively simple and doesn't really do much other than holding onto the L1 tokens approved by the Boss Bridge.

### <u>How Tokens Travel Between Layers</u>

- Tokens are sent to a vault on the *L1* , effectively locking them.
- A centralized off-chain service AKA **Boss Bridge**, signals the release of an equivalent number of tokens on the *L2*.
- Instead of directly transferring tokens from *L1* to *L2*, the tokens are locked on *L1* and an identical number of tokens are minted on the *L2* side.
- To transfer tokens back from *L2* to *L1*, the tokens are locked in a vault on the *L2* side.
- Centralized signers approve the unlocking of the original tokens on the *L1* side, completing the transfer process.


### <u>The Key Role of Signers</u>

So these Signers are important because they see who's depositing to either layer and decide when to unlock or relock tokens. As valuable as this function is, it is also an embedded known issue with the protocol due to its centralized nature.
Once a token in L1 gets locked in the vault, it's liberated to roam in L2. Reversibly, when you lock it back into the L2 vault, Signers get a signal, and the tokens from L1 vault are released.

**<u>Some of my personal notes [here](./.notes.md)**</u>
