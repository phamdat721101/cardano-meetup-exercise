# Cardano Basics Coding Exercise

## Exercise Title

_Getting Started with Cardano: Wallets, Transactions, and Native Assets_

---

## Overview

This exercise introduces you to Cardano fundamentals. You'll learn how to create wallets, send transactions, and interact with native assets on the Cardano blockchain.

_Objective: Understand core Cardano concepts by setting up a wallet, executing transactions, and exploring native tokens on the testnet._

---

## Prerequisites

- Cardano CLI installed ([Installation Guide](https://developers.cardano.org/docs/get-started/installing-cardano-node))
- Basic command line knowledge
- Access to Cardano Testnet
- Text editor for viewing transaction files

---

## Learning Objectives

- [ ] Understand Cardano's UTXO model
- [ ] Create and manage Cardano wallets
- [ ] Query blockchain data using Cardano CLI
- [ ] Send ADA transactions on testnet
- [ ] Explore native assets and tokens
- [ ] Understand transaction fees and structure

---

## Step-by-Step Instructions

### 1. Setup Environment

Verify your Cardano CLI installation:
```bash
cardano-cli --version
```

Set up testnet environment variables:
```bash
export CARDANO_NODE_SOCKET_PATH=/path/to/testnet/node.socket
export TESTNET_MAGIC=1  # Preview testnet
```

### 2. Create Your First Wallet

Generate payment key pair:
```bash
cardano-cli address key-gen \
    --verification-key-file payment.vkey \
    --signing-key-file payment.skey
```

Generate stake key pair:
```bash
cardano-cli stake-address key-gen \
    --verification-key-file stake.vkey \
    --signing-key-file stake.skey
```

Build your wallet address:
```bash
cardano-cli address build \
    --payment-verification-key-file payment.vkey \
    --stake-verification-key-file stake.vkey \
    --out-file payment.addr \
    --testnet-magic 1
```

View your address:
```bash
cat payment.addr
```

**Expected Output:** A Cardano address starting with `addr_test1...`

### 3. Fund Your Wallet

Get testnet ADA from the [Cardano Testnet Faucet](https://docs.cardano.org/cardano-testnet/tools/faucet/).

Query your wallet balance:
```bash
cardano-cli query utxo \
    --address $(cat payment.addr) \
    --testnet-magic 1
```

**Expected Output:**
```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
abc123...                                                    0        10000000000 lovelace
```

### 4. Send Your First Transaction

**Step 4.1:** Query protocol parameters:
```bash
cardano-cli query protocol-parameters \
    --testnet-magic 1 \
    --out-file protocol.json
```

**Step 4.2:** Create recipient address (or use another wallet):
```bash
# Recipient address (example)
RECIPIENT_ADDR=addr_test1qz...
```

**Step 4.3:** Build the transaction:
```bash
cardano-cli transaction build \
    --tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 1 | tail -1 | awk '{print $1"#"$2}') \
    --tx-out $RECIPIENT_ADDR+5000000 \
    --change-address $(cat payment.addr) \
    --testnet-magic 1 \
    --out-file tx.raw
```

**Step 4.4:** Sign the transaction:
```bash
cardano-cli transaction sign \
    --tx-body-file tx.raw \
    --signing-key-file payment.skey \
    --testnet-magic 1 \
    --out-file tx.signed
```

**Step 4.5:** Submit the transaction:
```bash
cardano-cli transaction submit \
    --tx-file tx.signed \
    --testnet-magic 1
```

**Expected Output:** `Transaction successfully submitted.`

**Step 4.6:** Verify transaction:
```bash
cardano-cli query utxo \
    --address $(cat payment.addr) \
    --testnet-magic 1
```

### 5. Explore Native Assets

Query tokens in your wallet:
```bash
cardano-cli query utxo \
    --address $(cat payment.addr) \
    --testnet-magic 1
```

**Discussion Points:**
- What are native assets on Cardano?
- How do they differ from ERC-20 tokens?
- What is the policy ID and asset name?

### 6. Understanding Transaction Structure

View transaction details:
```bash
cardano-cli transaction view --tx-file tx.signed
```

**Key concepts to identify:**
- Inputs (UTXOs)
- Outputs
- Transaction fees
- Metadata (if present)

---

## Collaborative Activity

**Group Exercise: Multi-Signature Transaction**

Break into pairs:
1. Each person creates a wallet
2. Share public addresses
3. Person A sends 10 ADA to Person B
4. Person B confirms receipt and sends 5 ADA back
5. Compare transaction times and fees

**Discussion:**
- How long did transactions take?
- What were the fee amounts?
- How does UTXO model differ from account-based models?

---

## Challenge Tasks (Optional)

For advanced participants:

1. **Create a Multi-Signature Address**: Build an address requiring 2-of-3 signatures
2. **Add Metadata**: Include JSON metadata in a transaction
3. **Mint a Native Token**: Create your own test token
4. **Script Automation**: Write a bash script to automate wallet creation

---

## Resources & References

- [Cardano Developer Portal](https://developers.cardano.org)
- [Cardano CLI Documentation](https://docs.cardano.org/development-guidelines/use-cli)
- [UTXO Model Explanation](https://docs.cardano.org/learn/eutxo-explainer)
- [Cardano Testnet Faucet](https://docs.cardano.org/cardano-testnet/tools/faucet/)
- [Cardano Explorer (Testnet)](https://preprod.cardanoscan.io)

---

## Troubleshooting

**Issue: "Connection refused" error**
- Verify `CARDANO_NODE_SOCKET_PATH` is correct
- Ensure Cardano node is running

**Issue: "Insufficient funds"**
- Check wallet balance with `query utxo`
- Get more testnet ADA from faucet
- Account for transaction fees (~0.17 ADA)

**Issue: "Invalid UTXO"**
- Query latest UTXOs before building transaction
- Ensure UTXO hasn't been spent already

---

## Solution Section (Add After Workshop)

Sample wallet creation script:
```bash
#!/bin/bash
# create_wallet.sh
cardano-cli address key-gen --verification-key-file payment.vkey --signing-key-file payment.skey
cardano-cli stake-address key-gen --verification-key-file stake.vkey --signing-key-file stake.skey
cardano-cli address build --payment-verification-key-file payment.vkey --stake-verification-key-file stake.vkey --out-file payment.addr --testnet-magic 1
echo "Wallet created! Address:"
cat payment.addr
```

---

## Feedback/Summary

**Key Takeaways:**
- Cardano uses the UTXO model for transactions
- Testnet is safe for experimentation
- Transaction fees are predictable and low
- Native assets are first-class citizens on Cardano

**Reflection Questions:**
1. How does Cardano's approach differ from other blockchains you've used?
2. What advantages does the UTXO model provide?
3. What challenges did you encounter?

---
