# ✨ So you want to sponsor a contest

This `README.md` contains a set of checklists for our contest collaboration.

Your contest will use two repos:

- **a _contest_ repo** (this one), which is used for scoping your contest and for providing information to contestants (wardens)
- **a _findings_ repo**, where issues are submitted (shared with you after the contest)

Ultimately, when we launch the contest, this contest repo will be made public and will contain the smart contracts to be reviewed and all the information needed for contest participants. The findings repo will be made public after the contest report is published and your team has mitigated the identified issues.

Some of the checklists in this doc are for **C4 (🐺)** and some of them are for **you as the contest sponsor (⭐️)**.

---

# Contest setup

## ⭐️ Sponsor: Provide contest details

Under "SPONSORS ADD INFO HERE" heading below, include the following:

- [ ] Create a PR to this repo with the below changes:
- [ ] Name of each contract and:
  - [ ] source lines of code (excluding blank lines and comments) in each
  - [ ] external contracts called in each
  - [ ] libraries used in each
- [ ] Describe any novel or unique curve logic or mathematical models implemented in the contracts
- [ ] Does the token conform to the ERC-20 standard? In what specific ways does it differ?
- [ ] Describe anything else that adds any special logic that makes your approach unique
- [ ] Identify any areas of specific concern in reviewing the code
- [ ] Add all of the code to this repo that you want reviewed

---

# Contest prep

## ⭐️ Sponsor: Contest prep

- [ ] Provide a self-contained repository with working commands that will build (at least) all in-scope contracts, and commands that will run tests producing gas reports for the relevant contracts.
- [ ] Make sure your code is thoroughly commented using the [NatSpec format](https://docs.soliditylang.org/en/v0.5.10/natspec-format.html#natspec-format).
- [ ] Modify the bottom of this `README.md` file to describe how your code is supposed to work with links to any relevent documentation and any other criteria/details that the C4 Wardens should keep in mind when reviewing. ([Here's a well-constructed example.](https://github.com/code-423n4/2021-06-gro/blob/main/README.md))
- [ ] Please have final versions of contracts and documentation added/updated in this repo **no less than 24 hours prior to contest start time.**
- [ ] Be prepared for a 🚨code freeze🚨 for the duration of the contest — important because it establishes a level playing field. We want to ensure everyone's looking at the same code, no matter when they look during the contest. (Note: this includes your own repo, since a PR can leak alpha to our wardens!)
- [ ] Promote the contest on Twitter (optional: tag in relevant protocols, etc.)
- [ ] Share it with your own communities (blog, Discord, Telegram, email newsletters, etc.)
- [ ] Optional: pre-record a high-level overview of your protocol (not just specific smart contract functions). This saves wardens a lot of time wading through documentation.
- [ ] Delete this checklist and all text above the line below when you're ready.

---

# Nouns DAO contest details

- $47,500 USDC main award pot
- $2,500 USDC gas optimization award pot
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-08-nounsdao-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts August 22, 2022 20:00 UTC
- Ends August 27, 2022 20:00 UTC

# About Nouns DAO

![](https://i.imgur.com/ZTZrWxw.png)

[Nouns](https://nouns.wtf) is a generative NFT project on Ethereum, where a new Noun is minted and auctioned off every day, all proceeds go to the DAO treasury, and each token represents one vote. To date more than 350 Nouns have been sold on auction, and the DAO has funded a variety of cool builders, including a recent 500 ETH grant to [theprotocolguild.eth](https://nouns.wtf/vote/108).

More intro information can be found on [Nouns Center](https://nouns.center/).

# Developer guide

## Setup

```sh
yarn
```

## Running tests

### Hardhat tests

```sh
yarn test
```

### Forge tests

```sh
forge install
forge test -vvv --ffi
```

# Audit scope

The focus of this audit is on:

- The [NounsDAOLogicV2](https://github.com/code-423n4/2022-08-nounsdao/blob/c1c7c6201d0247f92472419ff657b570f9104565/contracts/governance/NounsDAOLogicV2.sol) contract, which introduces two new features:
  - Dynamic quorum ([spec](https://github.com/nounsDAO/nouns-tech/tree/main/dynamic-quorum))
  - Voting gas refund ([spec](https://github.com/nounsDAO/nouns-tech/tree/main/vote-refund))
  - A fix to how `votingDelay` is used ([bug report](https://github.com/nounsDAO/nouns-diligence/blob/main/reports/proposal-58.md#major))
- Parent contracts which hold state variables: `NounsDAOStorageV2`, `NounsDAOStorageV1Adjusted`, `NounsDAOProxyStorage` (found in the file `NounsDAOInterfaces.sol`)
- The upgrade process from v1 to v2
  - The [NounsDAOLogicV1](https://github.com/code-423n4/2022-08-nounsdao/blob/c1c7c6201d0247f92472419ff657b570f9104565/contracts/governance/NounsDAOLogicV1.sol) contract for context
  - The upgrade will take place via a proposal; the proposal will include the following transactions:
    - call `_setImplementation` on the DAO proxy with the address of the V2 logic contract
    - call `_setDynamicQuorumParams` on the DAO proxy to set up dynamic quorum with parameters the DAO decided on
  - Proposals that were created on V1 and will remain active post-upgrade, should behave the same (e.g. maintain their quorum setting)
  - More generally, should ensure that there are no hidden storage issues when migrating from V1 → V2

Key risks we’d like you to explore:

- Bricking the DAO: this upgrade should not result in a non-functional DAO that cannot execute any additional proposals.
- Security: this upgrade should not introduce any new cost-effective attack vectors.

## Lines of code

| File                   | blank | comment | code |
| :--------------------- | ----: | ------: | ---: |
| NounsDAOLogicV2.sol    |   125 |     276 |  630 |
| NounsDAOLogicV1.sol    |    81 |     199 |  403 |
| NounsDAOInterfaces.sol |    66 |     164 |  210 |
| NounsDAOProxy.sol      |    18 |      56 |   67 |

- counted using [cloc](https://github.com/AlDanial/cloc)

# How to get context

- A basic understanding of [Compound’s Governor Bravo](https://compound.finance/docs/governance) is very useful, since Nouns DAO V1 is an NFT-adapted fork of it
- Read [NounsDAOLogicV1](https://github.com/code-423n4/2022-08-nounsdao/blob/c1c7c6201d0247f92472419ff657b570f9104565/contracts/governance/NounsDAOLogicV1.sol), since V2 is a diff from that; most importantly read [the docs articulating the modifications](https://github.com/code-423n4/2022-08-nounsdao/blob/c1c7c6201d0247f92472419ff657b570f9104565/contracts/governance/NounsDAOLogicV1.sol#L27) from Compound’s contract
- Get acquainted with [NounsToken](https://github.com/code-423n4/2022-08-nounsdao/blob/c1c7c6201d0247f92472419ff657b570f9104565/contracts/NounsToken.sol), and specifically the [ERC721Checkpointable](https://github.com/code-423n4/2022-08-nounsdao/blob/c1c7c6201d0247f92472419ff657b570f9104565/contracts/base/ERC721Checkpointable.sol) features
- For full context on how Nouns works (beyond the in-scope contracts), read the [Nouns Protocol wiki](https://www.notion.so/32e4f0bf74fe433e927e2ea35e52a507) that covers the auction house and on-chain art contracts
- Nice-to-have: read Dialectic’s two great posts on a Nouns governance attack on the treasury: [1](https://dialectic.ch/editorial/nouns-governance-attack), [2](https://dialectic.ch/editorial/nouns-governance-attack-2)

At this point you should have sufficient context to dive into V2:

- Read [the docs on V2’s modifications](https://github.com/code-423n4/2022-08-nounsdao/blob/c1c7c6201d0247f92472419ff657b570f9104565/contracts/governance/NounsDAOLogicV2.sol#L30)
- Read [the diff](https://gist.github.com/eladmallel/f8ab6b9e5a1bf664666a562b4f6429fd) between [`NounsDAOLogicV2`](https://github.com/code-423n4/2022-08-nounsdao/blob/c1c7c6201d0247f92472419ff657b570f9104565/contracts/governance/NounsDAOLogicV2.sol) and [`NounsDAOLogicV1`](https://github.com/code-423n4/2022-08-nounsdao/blob/c1c7c6201d0247f92472419ff657b570f9104565/contracts/governance/NounsDAOLogicV1.sol)
- Read the [e2e test](https://github.com/code-423n4/2022-08-nounsdao/blob/c1c7c6201d0247f92472419ff657b570f9104565/test/governance/NounsDAO/V2/end2end.test.ts#L255) that demos the upgrade to V2
