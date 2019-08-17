# Governance

Umbru Governance is a voting and funding platform. This documentation introduces and details the theory and practice to use the system.

### The Process

#### Introduction

* Governance consists of three components: Proposals, Votes, and Budgets
* Anyone can submit a proposal for a small fee
* Each valid masternode can vote for, against or abstain on proposals
* Approved proposals become budgets
* Budgets are paid directly from the blockchain to the proposal owner

#### Proposals

* Proposals are a request to receive funds
* Proposals can be submitted by anyone for a fee of 5 UMBRU. The proposal fee is irreversibly destroyed on submission.
* Proposals cannot be altered once submitted

#### Votes

* Votes are cast using the registered voting address
* The voting address can be delegated to a third party
* Votes can be changed at any time
* Votes are counted every 21600 blocks \(approx. 30 days\)

#### Budgets

* Budgets are proposals which receive a net total of yes votes equal to or greater than 10% of the total possible votes \(for example over 448 out of 4480\)
* Budgets can be nullified at any time if vote totals \(cast or re-cast\) fall below the approval threshold
* Budgets are processed \(paid\) in order of yes minus no votes. More popular budgets get payment priority.

#### Persistence

* Proposals become active one day after submission
* Proposals will remain visible on the network until they are either disapproved or the proposal’s last payment-cycle is reached
* Approval occurs when yes votes minus no votes equals 10% or more of the total available votes.
* Disapproval occurs when no votes minus yes votes equals 10% or more of the total available votes.
* The total available votes is the count of online and responding masternodes and can be seen by running the command `masternode count` in the Umbru Core wallet debug window. 

### Creating proposals

Once you have prepared your proposal and set up a website or post, it is time to submit your proposal to the blockchain for voting. While all tasks involved with creating a budget proposal can be executed from the Umbru Core wallet console, a tool providing a user interface have been developed to simplify this procedure.

* **Umbru Proposal Generator** [https://proposal.umbru.io](https://proposal.umbru.io)

### Voting on proposals

**You must vote at least three days before the superblock is created or your vote will not be counted. The exact deadline is 2160 blocks before the superblock.**

Voting on proposals is an important part of operating a masternode. Since masternodes are heavily invested in Umbru, they are expected to critically appraise proposals each month and vote in a manner they perceive to be consistent with the best interests of the network. Each masternode may vote once on each proposal, and the vote can be changed at any time before the voting deadline. 

#### Umbru Core wallet or masternode

If your Umbru Core wallet holds your Masternode voting addresses then you can vote from **Tools &gt; Debug console**. Each proposal will have a 'hash' to vode on. You will see a command for manual voting below. The voting code for Umbru Core Wallet is as follows \(yes/no/abstain\):

```text
gobject vote-many 6ed7418455e07f4b30b99f0d4a24a2b83282e12b26fe3415673ecbea04ff6c9d funding yes
gobject vote-many 6ed7418455e07f4b30b99f0d4a24a2b83282e12b26fe3415673ecbea04ff6c9d funding no
gobject vote-many 6ed7418455e07f4b30b99f0d4a24a2b83282e12b26fe3415673ecbea04ff6c9d funding abstain
```

Note this command will trigger a vote from all masternodes voting addresses held in your wallet. If you want to vote with only some of your masternodes, you must change the command from `vote-many` to `vote-alias`. If your vote was successful, you should see a confirmation message reading **Voted successfully**.

#### Delegating votes

Deterministic Masternode lists feature a separate voting key, which makes it possible to delegate your vote to a representative. Simply enter a Umbru address provided by the delegate when registering your masternode, or update your masternode registration to delegate the vote of a running masternode. The wallet controlling the private key to this address will then cast votes on behalf of this masternode owner simply by following the procedure described above. No further configuration is required.

