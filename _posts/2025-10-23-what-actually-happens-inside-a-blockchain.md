---
title: What Actually Happens Inside a Blockchain?
date: 2025-10-23 18:00:00 +1000
categories: [Web3 Basics]
tags: [web3, blockchain, decentralization]
---


## 1. Introduction

In the previous post we answered a simple question: [**What is blockchain?**]({% post_url 2025-10-22-what-is-blockchain %}) We learned that blockchain is like a **shared public notebook** that anyone can read, but no one can secretly change.

Now we go **behind the scenes** to understand:

- What happens when you send a blockchain transaction?
- Where does your transaction go?
- How is it approved and added to the blockchain?
- Why is blockchain secure even without banks or central authority?

---

## 2. Blockchain Works with 3 Key Ideas

To understand the process, keep these three core ideas in mind:

| Concept | Simple Meaning | Why It Matters |
|----------|----------------|----------------|
| **Decentralization** | No single company controls the network | No single point of failure |
| **Smart record-keeping** | Every transaction is stored forever | Clear and transparent history |
| **Trust through math** | Cryptography replaces middlemen | Secure without banks |

---

## 3. What Happens When You Make a Blockchain Transaction

Let’s walk through what really happens behind the scenes:

### ✅ Step-by-Step Journey

#### **Step 1: You Create a Transaction**
You enter the recipient’s wallet address and amount in your crypto wallet. Your **private key** digitally signs the transaction to prove it’s really you.

#### **Step 2: Transaction Is Broadcast**
Your wallet sends the signed transaction to nearby blockchain nodes.

#### **Step 3: Transaction Waits in the Mempool**
Nodes temporarily hold it in a waiting area called the **mempool**.

#### **Step 4: Miners or Validators Pick It Up**
A miner (Bitcoin) or validator (Ethereum) selects transactions from the mempool to include in the next block.

#### **Step 5: Block Is Verified**
Other nodes verify the block using blockchain rules.

#### **Step 6: Block Gets Added to the Chain**
Once verified, the block is linked to previous blocks — forming the blockchain.

#### **Step 7: Transaction Confirmed**
After the block is added, your transfer is complete and permanent.

---

## 4. Why Blockchain Is Secure

Here’s why tampering is nearly impossible:

- ✅ **Copies everywhere** – Thousands of computers store the blockchain
- ✅ **Cryptographic linking** – Each block is mathematically linked to the previous one
- ✅ **Rules + verification** – Invalid transactions are rejected by the network

---


## 5. The Importance of Consensus (Simple Version)

With thousands of computers running blockchain worldwide, how do they all **agree on a single version of the truth**?

They use a process called **consensus**.

Popular consensus mechanisms:

| Consensus Type | Used By | How It Works (Simple) |
|----------------|---------|------------------------|
| Proof of Work (PoW) | Bitcoin | Miners compete to solve puzzles |
| Proof of Stake (PoS) | Ethereum | Validators are selected based on stake |

> ✅ Consensus keeps everyone honest and prevents cheating like double spending.

---

## 6. Summary – 7 Steps of a Blockchain Transaction

| Step | Description |
|------|-------------|
| 1 | You create a transaction |
| 2 | It is shared with the network |
| 3 | It waits in the mempool |
| 4 | A block is created |
| 5 | The block is verified |
| 6 | It is added to blockchain |
| 7 | Transaction is confirmed |

