---
title: Wallet
katex: true
markup: "mmark"
---

To be backed by Bitcoin the federation needs a federated on-chain wallet. It is used to receive deposits that clients
exchange for blind tokens and to make payouts when clients want to withdraw. Generally it is just a multisig wallet
defined by a script descriptor. For example

```
wsh(sortedmulti(3,A,B,C,D))
```

where `A`, `B`, `C` and `D` are the public keys of the federation members.

Yet, notable differences exist. Other than most wallets we do not require extended public keys since we define our own
derivation scheme. Furthermore, the wallet can not rely on local fee estimation and the local chain tip since these
might be different for all members. Instead it uses the consensus algorithm to agree on these. Our particular protocol
will also use a randomness beacon.

## Chain tip consensus
To validate transactions a wallet needs to know the current chain tip. The problem with this is that different
federation members might see different chain tips either due to latency or even shallow forks.

To avoid the forking
problem we can just define that our internal chain tip is always e.g. 100 blocks behind the real one. This should
sufficiently mitigate the risk of being on different forks (bitcoin itself would be in trouble with such deep forks).
But the latency problem remains.

To solve this we use the consensus, each round each participant does the following:
1. Query `bitcoind` for the current block height
2. If the block height shrank, use the previous one
3. Propose `height - 100` as the consensus height
4. Receive peer proposals and use median as the new consensus height

Due to the assumption that less than $$\frac{1}{3}$$ of the participants are malicious, this will always leads to a
value that either was proposed by a honest participant or lies between two honest values to be chosen. Let's say that
all $$f$$ malicious proposals and $$n-2f$$ honest proposals are accepted, then $$f < n-2f$$ due to the previous
requirement. It is easy to see that the $$f$$ malicious proposals do not suffice to meaningfully alter the median.

Of course this assumes that all honest participants stay reasonable close to the real chain tip, but this is the task
of the operators and outside our protocol.

## Fee consensus
We also face a similar problem when spending Bitcoin. While the destinations and amounts are generally assumed to be
outputs of the consensus protocol and thus unproblematic one factor of transactions is not easily made deterministic:
the fees. But to avoid depletion attacks by overpaying fees we need to agree on them.

Naively we could use an algorithm that uses on-chain analysis to determine proper fee levels. But we only agree on
a tip buried 100 blocks deep, which would make the algorithm quite unresponsive. Furthermore other algorithms that take
the mempool into account may be preferable, but agreeing on the mempool is a fools errand. Instead we use a modified
version of the algorithm used for the chain tip consensus. Each round each participant does the following:

1. Query `bitcoind` for the current optimal fee rate
3. Propose said rate as the consensus fee rate
4. Receive peer proposals and use median as the new consensus height

The median argument works similarly and we achieve a honest consensus on fee rates.

## Randomness beacon
In some cases it is useful to have access to agreed-upon, fair randomness. Thus every round every participant also
proposes 32bytes of random data. The ones included in the consensus outcome are then XORed to form the round's
randomness beacon. We note that this is only safe if the items proposed to the consensus are encrypted till there is
agreement on which contributions will be included. This is the case for HBBFT. Otherwise an attacker could wait for the
other participants to announce their contribution and then adaptively chose his own to influence the outcome.

## Address Derivation
To allow clients to generate deposit addresses independent of the federation we do not use BIP32 derivation to
generate new addresses from the wallet descriptor, but a custom derivation scheme. We instead use a pay-to-contract
construction where a "contract" is hashed and added to all keys in the descriptor (added in the exponent in case of the
pub key). A descriptor with derived keys can then trivially be transformed into an address.

In MiniMint the "contract" is just a public key that can later be used to tie the deposit to a issuance transaction.

## Receiving Bitcoin
When depositing Bitcoin into the federation a client proceeds as follows:

1. Generate public/secret key pair
2. Tweak federation descriptor with public key
3. Send BTC to the resulting address
4. Generate [TxOutProof] and fetch raw transaction. These compact data structures allow the federation to verify the
   deposit with only the block hashes being synced and not the whole chain.
5. The tweak together with the TxOutProof and the raw transaction can now be sent to the federation to prove money was
   deposited. The federation should require a signature using the secret key.

Note that only once the federation is in possession of the tweak they can actually spend the funds as it is also needed
to tweak the private keys.

[TxOutProof]: https://bitcoincore.org/en/doc/0.21.0/rpc/blockchain/gettxoutproof/

## Sending Bitcoin
Once the federation agrees on paying Bitcoin to a set of destinations every participant deterministically selects
the necessary outputs. The previously agreed-upon fee rate is used to determine the fee. In case a change address is needed
the randomness beacon is used to derive a random change address just as a deposit address would be derived.

This transaction is then signed by each participant individually and the signatures broadcasted via the consensus
protocol. Note that due to the transaction being generated deterministically it does not need to be exchanged itself.

After receiving sufficient signatures each party can assemble the final transaction and broadcast it.