+++

title = "sigstore, blockchain vs transparency logs"
date = "2022-05-30"
tags = ["sigstore","blockchain","devsecops"]
draft = false
author = "Luke Hinds"
type = "post"

+++

Co-authored by Luke Hinds (Red Hat) and Prof Santiago Torres-Arias (Purdue University).

> **Disclaimer:** The following is representative of the authors views, and not necessarily that of the sigstore community.

***“Why did you chose to use a transparency log and not a blockchain?”\***

We get this question often, so we figured it’s best to address this head on. It could be best summarised as *‘****why would we use blockchain?\****’* as opposed to ‘***why did we not use blockchain?\***’. The following points should illustrate how the first question is more apt.

### Tokens & Fees

The first issue is the need for a token and its related fees.

For a public blockchain a token of some sort is required to transact on the chain. A majority of the time these tokens are acquired as part of an initial ‘giveaway’, conversion of fiat money via some sort of exchange or via the use of compute power (in the case of proof-of-work).

With sigstore the key tenet of the service is that it is ‘***free\***’ at point of use. We want everyone to be able to sign and get the same level of service, no matter how many resources you have at your disposal (both fiscal or computational). This is not possible when a token is involved. Typically fees are required and a general trend exists where the more popular the chain ecosystem becomes, the higher the token value inflates.

![](/images/tweet.png)

With a transparency log, no tokens / fees are required. It’s an API with a merkle tree as a backend. If you have an internet connection, you can sign artifacts and make records.

***“How about Hyperledger Fabric / R3 Corda? Those are token-less?”\***. True, but they have a private / “permissioned” mode of operation (there is often a need for additional access control layers, which often ends up negating the decentralised claim).

There are also considerations of factors such as a 51% attack. We do not want to have to be concerned about someone having majority ownership of a token / or consolidated hashing power.

### Private Key (Wallet) management

A key area (excuse the pun) that sigstore was devised to address, is the lack of cryptographic signing adoption due to the pains of key management. The truth is the majority do not want to deal with the headache and cost of storing private keys securely. This is why cryptographic signing adoption is so low in open source communities (typically lower than 5%). With blockchain a key pair is required (a wallet). This is a problem for the following reasons:

- OSS contributors are often unfunded and not employed to develop open source. Quite often they could be from a developing nation or somehow disadvantaged financially (most students right?!). It is not fair to expect them to spend upwards of $100 on buying a hardware wallet.
- Likewise we cannot expect users to store their wallet and passphrase in a cloud service.

There are other alternatives using something like MetaMask, but we get into that a little later.

### Controversy

This one is perhaps not fair, but alas it comes as a package with blockchain.

In sigstore, the more who adopt cryptographic signing, the better it is for all (a more secure supply chain). The truth is that blockchain makes for a controversial topic. Post to any forum about blockchain (save the ones catering to those holding crypto-currency) and you will find a very flammable mix of views. One group feels it is the answer to everything, the other feels it’s no more than a Ponzi scheme. Those in the middle are typically undecided and often drowned out.

We don’t have any of this concern with a transparency log. It’s a largely neutral technology.

### Production Ready

Sigstore uses an existing implementation that has been put through its paces in production. We use the same backend as [certificate-transparency](https://certificate.transparency.dev/howctworks/).

Certificate transparency logs are queried millions of times a day by browsers to ensure a public transparent chain exists for all TLS certificates. The truth is we would have been in a ‘prototype’ stage for a lot longer if using blockchain, while we gained the confidence to have sigstore run in production. And of course, as said, we would have needed to address all of the above points too.

### Decentralisation

One clear advantage a blockchain has is its distributed / decentralised architecture. However when we dig into the current developer landscape we often find its not decentralised. As mentioned under the Token heading, permissioned / token-less blockchains require gateways for access-control , canonicalization etc. dApps use centralised API access from providers such as [Infura](https://infura.io/) or [Alchemy](https://www.alchemy.com/) to interact with the blockchain.

MetaMask the most popular wallet implementation used by clients to communicate with Ethereum, also uses the Infura central API.

![](/images/infura.png)

After the[ DAO hack in ethereum](https://www.gemini.com/cryptopedia/the-dao-hack-makerdao), a hard fork by developers created wide controversy about the other “angles” of centralisation that may exist: in this case, the fact that a series of developers can — and have — effectively thrown away the protocol-level decentralisation.

### **Would Sigstore Ever Use Blockchain?**

It is possible! There is nothing forcing us to use one technology over the other. Instead, we are looking for properties that we can use today to help us secure the software ecosystem. While our assessment is that blockchains do not fit the bill (now), we wouldn’t discard a constantly-evolving and constantly-innovating field forever. Likewise, we are an open source community, which means we are always happy to hear proposals, weight them by their technical merit and the impact that they could have for everybody. If you feel that there are particular blockchains that could play with, we strongly encourage you to drop by the community meeting and show us how such a system would work — demos are very welcome.