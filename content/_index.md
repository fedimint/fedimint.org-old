# Welcome to Fedimint.org

This site is intended to collect research and ideas about federated chaumian mints (or federated chaumian banks, but
nobody likes banks) to scale Bitcoin while also making it more private.

## Chaumian Mints
One of the (if not the) earliest e-cash schemes were [Chaumian mints or banks]. They use blind signatures to allow the
anonymous transfer of backing assets held by the mint. The basic idea is that a user can give the mint some amount x of
an asset and the mint in turn blind signs x IOUs that allow the user to either withdraw the asset or exchange
them for new IOUs or products. The small word "blind" does the heavy lifting here, it means that the user and mint run
a cryptographic protocol that allows the user to acquire a digital signature on some data without the mint learning 
anything about the message or the signature so that when the mint sees one of its signatures for some message it can
no longer tell to whom it was issued. This means trading these IOUs is completely anonymous.

"Why have I never heard of this before?" you may ask and the sad answer is that it never really caught on. One big
problem of chaumian mints is that they are a single point of failure and an easy target for regulation and other
attacks. Most countries financial regulations disallow anonymous payments to some degree, so running a mint in the open
is a bad idea. Running one anonymously brings with it the problem of trust, the operator could run away with the money
at any point. This combination of problems relegated the concept to a very small, low value market, e.g.
[to pay for watchtower fees] in lightning. 

[Chaumian mints or Banks]: http://www.hit.bme.hu/~buttyan/courses/BMEVIHIM219/2009/Chaum.BlindSigForPayment.1982.PDF
[to pay for watchtower fees]: https://lightning-wallet.com/storage-tokens#storage-tokens

## What is a Federated Mint?
In a federated mint the required trust is split over multiple parties. It employs a consensus algorithm
and threshold cryptography to guarantee both safety and availability even in the case that some parties are
compromised. That means if the participants are sufficiently distributed not even a nation state level attacker can
harm the federation. Running it anonymously also becomes much more attractive since there is no single party anymore
that could run away with the backing funds.

## How does it relate to Bitcoin
Bitcoin is the first asset in human history that can truly be held in a federated manner, meaning only being accessible
if a certain quorum of people agrees. It is thus the perfect backing asset for a federated mint. A first, primitive
version could work as follows:

* **deposit**: A User sends BTC to the federation's wallet and in turn receives the corresponding amount of tokens.
* **transfer**: The user can then pay someone else using these tokens, which works as follows:
  * The payer selects appropriately many tokens and sends them to the payee
  * The payee exchanges these tokens for new ones using the federated mint
  * Only then the payee accepts the transaction, as the exchange may fail in case of a double spend
* **withdraw**: Finally any user can redeem tokens for BTC again.

We see that between deposit and withdrawal there can be many internal transactions, so federated mints do not only provide
excellent privacy but also scaling. One problem with this primitive version is the enormous centralization pressure it
exerts, as two federated mints won't accept each other's tokens, making big mints more attractive. This can be mitigated
by integrating with Lightning. For this the federation needs to support two more operations:

* **ln-send**: pay an LN invoice using tokens.
* **ln-receive**: issue an invoice to a user. Once it is paid the user receives the appropriate amount of tokens.

With these two operations any federated mint suddenly becomes interoperable with any other Lightning node, including
other federated mints. The federation essentially becomes a hosted but federated Lightning wallet.

## What about centralization
Yes, a federated mint requires more trust than a self-hosted Lighning node or on-chain Bitcoin. But we think that the
risk can be minimized sufficiently by distributing the federation members. There are many people that can not or do not
want to run their own lightning wallet, be it because of fees or maintenance effort. For these a federated mint is
much preferable to centralized solutions as it protects user privacy and has no single points of failure. Systemic risks
should be sufficiently mitigated by the fact that any willing group of people should be able to start their own
federation.

## Existing projects
We are currently aware of two efforts to build such a federated mint:
* [MiniMint] (to be renamed to *Fedimint* [^1]): A modular federated e-cash prototype still under heavy development written in Rust. It already supports all main operations (deposit/withdraw via both on-chain Bitcoin and Lightning, e-cash transfers) and comes with a rudimentary CLI client. Some features are still missing and blocking mainnet deployments, but the project is moving quickly. If you are interested in contributing [check out the GitHub repository](https://github.com/fedimint/minimint).

## Prior art
* [SCRIT1]: A half-finished implementation of a federated chaumian mint written in Go, developed by Frank Braun and Jonathan Logan.
  It does not implement BTC backing, but was the first public implementation.
* SCRIT2: A reimplementation of a federated chaumian mint written in Go, supporting multiple currencies, inter-currency swap transactions, receiver- and sender-initiated half-offline transactions as well as complex multiparty transactions. It is in private beta and has no direct linkage with bitcoin yet.
  You can read more on [Jonathan's blog].

[Open Transactions] also deserves a honorable mention since it already allowed for the issuance of e-cash tokens backed
by Bitcoin held in a multisig wallet. It does not appear to support threshold issuance of e-cash tokens though.

[SCRIT1]: https://github.com/scritcash
[Jonathan's blog]: https://opaque.link/post/scrit-vision/
[MiniMint]: https://github.com/fedimint/minimint
[Open Transactions]: https://www.opentransactions.org/wiki/Main_Page

## Resources
* [Q&A](https://docs.google.com/document/d/1ZLjWmczADUhCsaRjE2ta_8BbgqPxPhabzxhLEsUlmoo)
* [An interview] from [HCPP21] with smuggler and elsirion about both SCRIT and MiniMint.
* [Blockstream blogpost] announcing sponsorship
* Bitcoin Magazine [Twitter space] that gives different perspectives
* [A talk about MiniMint] at Adopting Bitcoin 2021
* [Citadel Dispatch e45]
* [SLP331] Eric Sirion – MiniMint, Federated Mints for Bitcoin scaling and privacy
* [Bitcoin Explained with Ruben Somsen] about federated e-cash
* [Marty Bent's review of Federated Chaumian Mints] overviewing Obi Nwosu's and Eric Sirion's talks at Bitcoin Miami 2022

Please feel free to [open PRs] for corrections and additions.

[HCPP21]: https://chaos.hcpp.cz/
[an interview]: https://www.youtube.com/watch?v=JXGmzTbyuEw&t=5330s
[Blockstream blogpost]: https://medium.com/blockstream/blockstream-sponsors-federated-e-cash-as-a-bitcoin-scaling-technology-637ba05de7b3
[Twitter space]: https://www.youtube.com/watch?v=A_7-DsreUQg
[A talk about MiniMint]: https://bitcointv.com/w/kHwmbLTWjsbaDTJpBewUmX
[Citadel Dispatch e45]: https://bitcointv.com/w/uTtKtUmfWZ7mZDztVXVRWv
[SLP331]: https://stephanlivera.com/episode/331/
[Bitcoin Explained with Ruben Somsen]: https://www.youtube.com/watch?v=alyYNIX0m3o
[open PRs]: https://github.com/fedimint/fedimint.org
[Marty Bent's review of Federated Chaumian Mints]: https://bitcoinmagazine.com/technical/chaumian-mints-distribute-trust-among-bitcoin-users

## Support and Donations
The Fedimint project is grateful for the generous support and donations we've received from various organisations and individuals including:

* [Blockstream]
* [Obi Nwosu]
* [Einundzwanzig]
* [Human Rights Foundation]
* [Ten 31]
* [Spiral]

[Blockstream]: https://blockstream.com/
[Obi Nwosu]: https://twitter.com/obi
[Einundzwanzig]: https://einundzwanzig.space/
[Human Rights Foundation]: https://hrf.org/
[Ten 31]: https://ten31.vc/
[Spiral]: https://spiral.xyz/

[^1]: Initially *Fedimint* was meant to be an umbrella organization for the different efforts to build federated Chaumian e-cash mints. Unfortunately only one project, *MiniMint*, generated any traction and continues to be actively developed. That's why *Fedimint* became synonymous with *MiniMint* over time and today the two names are a source of much confusion. A renaming of *MiniMint will involve many code changes, that's why it hasn't happened yet.
