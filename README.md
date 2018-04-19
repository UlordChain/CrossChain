# CrossChain
**On-chain atomic swaps for Ulord and other cryptocurrencies. **
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

##Theory

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

