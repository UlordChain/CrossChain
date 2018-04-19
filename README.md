# CrossChain
**On-chain atomic swaps for Ulord and other cryptocurrencies.**

[中文版说明](https://github.com/UlordChain/CrossChain/blob/master/README_CN.md)



[TOC]



## TODO（5%）

### command line
- [ ] Initiate

- [ ] participate

- [ ] redeem

- [ ] refund

- [ ] extractsecret

- [ ] auditcontract

### Script Operation
- [ ] OP_CHECKLOCKTIMEVERIFY

### Script
- [ ] func CrossChainContract
- [ ] func redeemP2SHContract
- [ ] func refundP2SHContract
### RPC Interface
- [ ] Ulord RPCclient
### Doc
- [ ] SequenceDiagram
- [x] README
- [ ] README_CN

## Note
These tools do not operate solely on-chain. A side-channel is required between each party performing the swap in order to exchange additional data. This side-channel could be as simple as a text chat and copying data. Until a more streamlined implementation of the side channel exists, such as the Lightning Network, these tools suffice as a proof-of-concept for cross-chain atomic swaps and a way for early adopters to try out the technology.

Due to the requirements of manually exchanging data and creating, sending, and watching for the relevant transactions, it is highly recommended to read this README in its entirety before attempting to use these tools. The sections below explain the principles on which the tools operate, the instructions for how to use them safely, and an example swap between Ulord Chain and second chain.

**your chain can be used so long as it supports OP_SHA256 and OP_CHECKLOCKTIMEVERIFY.**

## Theory

A cross-chain swap is a trade between two users of different cryptocurrencies. For example, one party may send Ulord to a second party's Ulord address, while the second party would send Bitcoin to the first party's Bitcoin address. However, as the blockchains are unrelated and transactions can not be reversed, this provides no protection against one of the parties never honoring their end of the trade. One common solution to this problem is to introduce a mutually-trusted third party for escrow. An atomic cross-chain swap solves this problem without the need for a third party.

Atomic CrossChain swaps involve each party paying into a contract transaction, one contract for each blockchain. The contracts contain an output that is spendable by either party, but the rules required for redemption are different for each party involved.

One party (called counterparty 1 or the initiator) generates a secret and pays the intended trade amount into a contract transaction. The contract output can be redeemed by the second party (called countryparty 2 or the participant) as long as the secret is known. If a period of time ( **typically 48 hours**) expires after the contract transaction has been mined but has not been redeemed by the participant, the contract output can be refunded back to the initiator's wallet.

For simplicity, we assume the initiator wishes to trade Bitcoin for Ulord with the participant. The initiator can also trade Ulord for Bitcoin and the steps will be the same, but with each step performed on the other blockchain.

The participant is unable to spend from the initiator's Bitcoin contract at this point because the secret is unknown by them. If the initiator revealed their secret at this point, the participant could spend from the contract without ever honoring their end of the trade.

The participant creates a similar contract transaction to the initiator's but on the Ulord blockchain and pays the intended Ulord amount into the contract. However, for the initiator to redeem the output, their own secret must be revealed. For the participant to create their contract, the initiator must reveal not the secret, but a cryptographic hash of the secret to the participant. The participant's contract can also be refunded by the participant, but only after half the period of time that the initiator is required to wait before their contract can be refunded ( **typically 24 hours**).

With each side paying into a contract on each blockchain, and each party unable to perform their refund until the allotted time expires, the initiator redeems the participant's Ulord contract, thereby revealing the secret to the participant. The secret is then extracted from the initiator's redeeming Ulord transaction providing the participant with the ability to redeem the initiator's Bitcoin contract.

This procedure is atomic (with timeout) as it gives each party at least 24 hours to redeem their coins on the other blockchain before a refund can be performed.

The image below provides a visual of the steps each party performs and the transfer of data between each party.

![SequenceDiagram](https://github.com/UlordChain/CrossChain/blob/master/CrossChain%E2%80%98s%20SequenceDiagram_EN.jpg)

## Command Line
Separate command line utilities are provided to handle the transactions required to perform a cross-chain atomic swap for each supported blockchain. For a swap between Bitcoin and Ulord, the two utilities btcatomicswap and dcratomicswap are used. Both tools must be used by both parties performing the swap.

Different tools may require different flags to use them with the supported wallet. For example, btcatomicswap includes flags for the RPC username and password while Ulordswap does not. Running a tool without any parameters will show the full usage help.

All of the tools support the same six commands. These commands are:

  ```
Commands:

  initiate <participant address> <amount>
  participate <initiator address> <amount> <secret hash>
  redeem <contract> <contract transaction> <secret>
  refund <contract> <contract transaction>
  extractsecret <redemption transaction> <secret hash>
  auditcontract <contract> <contract transaction>
  ```
**`initiate <participant address> <amount>`**

The initiate command is performed by the initiator to create the first contract. The contract is created with a locktime of 48 hours in the future. This command returns the secret, the secret hash, the contract script, the contract transaction, and a refund transaction that can be sent after 48 hours if necessary.

Running this command will prompt for whether to publish the contract transaction. If everything looks correct, the transaction should be published. The refund transaction should be saved in case a refund is required to be made later.

**`participate <initiator address> <amount> <secret hash>`**

The participate command is performed by the participant to create a contract on the second blockchain. It operates similarly to initiate but requires using the secret hash from the initiator's contract and creates the contract with a locktime of 24 hours.

Running this command will prompt for whether to publish the contract transaction. If everything looks correct, the transaction should be published. The refund transaction should be saved in case a refund is required to be made later.

**`redeem <contract> <contract transaction> <secret>`**

The redeem command is performed by both parties to redeem coins paid into the contract created by the other party. Redeeming requires the secret and must be performed by the initiator first. Once the initiator's redemption has been published, the secret may be extracted from the transaction and the participant may also redeem their coins.

Running this command will prompt for whether to publish the redemption transaction. If everything looks correct, the transaction should be published.

**`refund <contract> <contract transaction>`**

The refund command is used to create and send a refund of a contract transaction. While the refund transaction is created and displayed during contract creation in the initiate and participate steps, the refund can also be created after the fact in case there was any issue sending the transaction (e.g. the contract transaction was malleated or the refund fee is now too low).

Running this command will prompt for whether to publish the redemption transaction. If everything looks correct, the transaction should be published.

**`extractsecret <redemption transaction> <secret hash>`**

The extractsecret command is used by the participant to extract the secret from the initiator's redemption transaction. With the secret known, the participant may claim the coins paid into the initiator's contract.

The secret hash is a required parameter so that "nonstandard" redemption transactions won't confuse the tool and the secret can still be discovered.

**`auditcontract <contract> <contract transaction>`**

The auditcontract command inspects a contract script and parses out the addresses that may claim the output, the locktime, and the secret hash. It also validates that the contract transaction pays to the contract and reports the contract output amount. Each party should audit the contract provided by the other to verify that their address is the recipient address, the output value is correct, and that the locktime is sensible.
