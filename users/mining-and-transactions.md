# Mining and Transactions

## Mining

Mining in the context of cryptocurrency such as Umbru refers to the process of searching for solutions to cryptographically difficult problems as a method of securing blocks on the blockchain. The process of mining creates new currency tokens as a reward to the miner. Mining is possible on a range of hardware. Umbru implements an algorithm known as X25X, which the miner must solve in order to earn rewards.

The simplest and most general hardware available for mining is the general purpose CPU present in every computer. A CPU is designed to be versatile but offers less efficiency than a GPU, which is designed to rapidly calculate millions of vectors in parallel. While specific CPU instruction enhancements related to cryptography such as AES or AVX can provide a decent boost, GPUs offer a significant performance increase due to their multiple pipelines capable of processing the predictably repetitive calculations associated with cryptocurrency mining.

### Mining Pools

* **Official Pool** [https://pool.umbru.io](https://pool.umbru.io)

### Masternodes vs. Mining

Umbru, like Bitcoin and most other cryptocurrencies, is based on a decentralized ledger of all transactions, known as a blockchain. This blockchain is secured through a consensus mechanism; in the case of both Dash and Bitcoin, the consensus mechanism is Proof of Work \(PoW\). Miners attempt to solve difficult problems with specialized computers, and when they solve the problem, they receive the right to add a new block to the blockchain. If all the other people running the software agree that the problem was solved correctly, the block is added to the blockchain and the miner is rewarded.

Umbru works a little differently from Bitcoin, however, because it has a two-tier network. The second tier is powered by Masternodes, which enable network functions, and the decentralized governance and budget system. Because this second tier is so important, masternodes are also rewarded when miners discover new blocks. The breakdown is as follows: 70% of the block reward goes to the miner, 20% goes to masternodes, and 10% is reserved for the budget system \(created by superblocks every month\).

The masternode system is referred to as Proof of Service, since the masternodes provide crucial services to the network. In fact, the entire network is overseen by the masternodes, which have the power to reject improperly formed blocks from miners.

In short, miners power the first tier, which is the basic sending and receiving of funds and prevention of double spending. Masternodes power the second tier, which provide the added features that make Umbru different from other cryptocurrencies. Masternodes do not mine, and mining computers cannot serve as masternodes. Additionally, each masternode is “secured” by 5000 UMBRU. Those funds remain under the sole control of their owner at all times, and can still be freely spent. The funds are not locked in any way. However, if the funds are moved or spent, the associated masternode will go offline and stop receiving rewards.

