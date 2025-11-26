# Hydra Layer 2 Exercise - Cardano Meetup

## Table of Contents
1. [Introduction](#introduction)
2. [What is Cardano?](#what-is-cardano)
3. [What is Hydra Layer 2?](#what-is-hydra-layer-2)
4. [Prerequisites](#prerequisites)
5. [Environment Setup](#environment-setup)
6. [Hands-On Exercise: Setting Up Hydra](#hands-on-exercise-setting-up-hydra)
7. [Sample Workflow](#sample-workflow)
8. [Group Activity](#group-activity)
9. [Troubleshooting](#troubleshooting)
10. [Additional Resources](#additional-resources)

---

## Introduction

Welcome to the Hydra Layer 2 hands-on exercise! In this lab, you'll learn about Cardano's innovative scaling solution, Hydra, and get practical experience setting up Hydra nodes, opening a Head, and performing fast off-chain transactions.

**Time Required:** 60-90 minutes  
**Difficulty Level:** Intermediate  
**Group Size:** 2-5 participants recommended

### Important Disclaimer

This document is designed for **educational purposes** and provides a conceptual overview of Hydra Layer 2 technology. Some examples and API calls are simplified for learning clarity. For production implementations:

- Always consult the [official Hydra documentation](https://hydra.family/head-protocol/)
- Use the actual API specifications for your Hydra version
- Test thoroughly on testnets before any mainnet deployment
- Understand the security implications and requirements

The Hydra protocol and API continue to evolve. Always refer to the latest official documentation for accurate implementation details.

---

## What is Cardano?

**Cardano** is a proof-of-stake blockchain platform founded on peer-reviewed research and developed through evidence-based methods. Key features include:

- **Sustainability**: Energy-efficient consensus mechanism (Ouroboros)
- **Scalability**: Multi-layered architecture designed for scaling
- **Interoperability**: Built with cross-chain communication in mind
- **Security**: Formal verification and peer-reviewed protocols

Cardano processes transactions on its mainnet (Layer 1), but as usage grows, there's a need for faster, more cost-effective solutions—this is where **Layer 2** solutions like Hydra come in.

---

## What is Hydra Layer 2?

**Hydra** is Cardano's Layer 2 scalability solution that enables:

- **Fast Transactions**: Near-instant confirmation times (milliseconds)
- **Low Fees**: Minimal transaction costs within a Hydra Head
- **High Throughput**: Process thousands of transactions per second
- **Security**: Inherits security from Cardano Layer 1

### How Hydra Works

Hydra uses the concept of **"Heads"** - off-chain mini-ledgers where participants can:

1. **Open a Head**: Multiple parties commit funds to a shared state
2. **Transact Off-Chain**: Execute transactions within the Head at high speed
3. **Close the Head**: Final state is settled back to Layer 1

Think of a Hydra Head as a secure, temporary payment channel between parties where they can transact freely before finalizing everything on the main chain.

### Use Cases

- Payment channels for instant micropayments
- Gaming applications requiring fast state updates
- DeFi protocols with high transaction volumes
- Multi-party state channels for collaborative applications

---

## Prerequisites

Before starting this exercise, ensure you have:

### System Requirements

- **OS**: Linux (Ubuntu 20.04+), macOS (11+), or WSL2 on Windows
- **RAM**: At least 4GB available
- **Storage**: 10GB free space
- **CPU**: 2+ cores recommended

### Software Requirements

1. **Docker** (recommended for beginners)
   - Docker Engine 20.10+
   - Docker Compose 1.29+

2. **OR Native Binaries** (for advanced users)
   - Cardano Node 8.0+
   - GHC 8.10.7 or 9.2.8
   - Cabal 3.6+

3. **Additional Tools**
   - `curl` or `wget`
   - `jq` (JSON processor)
   - `git`
   - Text editor (VS Code, vim, nano)

### Knowledge Prerequisites

- Basic understanding of blockchain concepts
- Command-line interface familiarity
- Understanding of Cardano transactions (helpful but not required)

---

## Environment Setup

### Option 1: Docker Setup (Recommended)

Docker provides the easiest way to get started with Hydra without complex dependencies.

#### Step 1: Install Docker

**Ubuntu/Debian:**
```bash
# Update package index
sudo apt-get update

# Install Docker
sudo apt-get install -y docker.io docker-compose

# Add your user to docker group (logout/login after this)
sudo usermod -aG docker $USER

# Verify installation
docker --version
docker-compose --version
```

**macOS:**
```bash
# Install Docker Desktop from https://www.docker.com/products/docker-desktop
# Or use Homebrew:
brew install --cask docker

# Verify installation
docker --version
docker-compose --version
```

#### Step 2: Pull Hydra Docker Images

```bash
# Pull a specific Hydra node image version (recommended for stability)
docker pull ghcr.io/input-output-hk/hydra-node:0.15.0

# Verify the image
docker images | grep hydra-node
```

**Expected Output:**
```
ghcr.io/input-output-hk/hydra-node   0.15.0    abc123def456   2 days ago    500MB
```

**Note:** Using a specific version tag ensures consistent behavior across all participants.

#### Step 3: Set Up Working Directory

```bash
# Create a workspace for Hydra
mkdir -p ~/hydra-exercise
cd ~/hydra-exercise

# Create necessary directories
mkdir -p {node1,node2,node3}/{keys,state,logs}
```

### Option 2: Native Binary Setup (Advanced)

For users who prefer native binaries or want better performance:

#### Step 1: Install System Dependencies

**Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install -y \
  build-essential \
  libffi-dev \
  libgmp-dev \
  libssl-dev \
  libtinfo-dev \
  libsystemd-dev \
  zlib1g-dev \
  make \
  g++ \
  git \
  jq \
  curl
```

**macOS:**
```bash
brew install libsodium
brew install openssl
brew install jq
```

#### Step 2: Install Hydra Node Binary

```bash
# Download the latest Hydra release
HYDRA_VERSION="0.15.0"  # Check for latest version
wget https://github.com/input-output-hk/hydra/releases/download/${HYDRA_VERSION}/hydra-x86_64-linux.zip

# Extract the archive
unzip hydra-x86_64-linux.zip

# Move binaries to PATH
sudo mv hydra-node hydra-tui /usr/local/bin/

# Verify installation
hydra-node --version
```

**Expected Output:**
```
hydra-node 0.15.0 - git revision abc123
```

#### Step 3: Set Up Cardano Node (Required for Native)

```bash
# Download and install cardano-node
# Follow official Cardano documentation:
# https://developers.cardano.org/docs/get-started/installing-cardano-node/

# For this exercise, we'll use a lightweight preview testnet setup
```

---

## Hands-On Exercise: Setting Up Hydra

Now let's walk through setting up a local Hydra network with multiple nodes!

### Exercise Architecture

We'll create a 3-node Hydra network:
- **Node 1 (Alice)**: First participant
- **Node 2 (Bob)**: Second participant  
- **Node 3 (Carol)**: Third participant

Each node will be able to open a Head together and transact off-chain.

### Step 1: Generate Keys for Participants

Each Hydra participant needs cryptographic keys.

```bash
cd ~/hydra-exercise

# Generate keys for Alice (Node 1)
docker run --rm -v $(pwd)/node1/keys:/keys \
  ghcr.io/input-output-hk/hydra-node:0.15.0 \
  gen-hydra-key --output-file /keys/hydra

# Generate keys for Bob (Node 2)
docker run --rm -v $(pwd)/node2/keys:/keys \
  ghcr.io/input-output-hk/hydra-node:0.15.0 \
  gen-hydra-key --output-file /keys/hydra

# Generate keys for Carol (Node 3)
docker run --rm -v $(pwd)/node3/keys:/keys \
  ghcr.io/input-output-hk/hydra-node:0.15.0 \
  gen-hydra-key --output-file /keys/hydra

# Verify keys were created
ls -la node*/keys/
```

**Expected Output:**
```
node1/keys/:
-rw------- 1 user user  hydra.sk
-rw-r--r-- 1 user user  hydra.vk

node2/keys/:
-rw------- 1 user user  hydra.sk
-rw-r--r-- 1 user user  hydra.vk

node3/keys/:
-rw------- 1 user user  hydra.sk
-rw-r--r-- 1 user user  hydra.vk
```

> **Note**: `.sk` files are signing keys (keep private!), `.vk` files are verification keys (public).

### Step 2: Create Network Configuration

Create a Docker Compose file to orchestrate the Hydra network:

**Note:** This is a simplified demo configuration. For a production setup, you'll need to:
1. Provide proper Cardano node connection details
2. Mount protocol parameters file
3. Configure cardano-node integration

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  hydra-node-1:
    image: ghcr.io/input-output-hk/hydra-node:0.15.0
    container_name: alice
    ports:
      - "4001:4001"  # API port
      - "5001:5001"  # Hydra network port
    volumes:
      - ./node1/keys:/keys:ro
      - ./node1/state:/state
      - ./node1/logs:/logs
    command: >
      --node-id alice
      --api-host 0.0.0.0
      --api-port 4001
      --host 0.0.0.0
      --port 5001
      --peer hydra-node-2:5002
      --peer hydra-node-3:5003
      --hydra-signing-key /keys/hydra.sk
      --hydra-verification-key /keys/hydra.vk
    networks:
      - hydra-net

  hydra-node-2:
    image: ghcr.io/input-output-hk/hydra-node:0.15.0
    container_name: bob
    ports:
      - "4002:4002"
      - "5002:5002"
    volumes:
      - ./node2/keys:/keys:ro
      - ./node2/state:/state
      - ./node2/logs:/logs
    command: >
      --node-id bob
      --api-host 0.0.0.0
      --api-port 4002
      --host 0.0.0.0
      --port 5002
      --peer hydra-node-1:5001
      --peer hydra-node-3:5003
      --hydra-signing-key /keys/hydra.sk
      --hydra-verification-key /keys/hydra.vk
    networks:
      - hydra-net

  hydra-node-3:
    image: ghcr.io/input-output-hk/hydra-node:0.15.0
    container_name: carol
    ports:
      - "4003:4003"
      - "5003:5003"
    volumes:
      - ./node3/keys:/keys:ro
      - ./node3/state:/state
      - ./node3/logs:/logs
    command: >
      --node-id carol
      --api-host 0.0.0.0
      --api-port 4003
      --host 0.0.0.0
      --port 5003
      --peer hydra-node-1:5001
      --peer hydra-node-2:5002
      --hydra-signing-key /keys/hydra.sk
      --hydra-verification-key /keys/hydra.vk
    networks:
      - hydra-net

networks:
  hydra-net:
    driver: bridge
EOF
```

**Important Notes:**
- This configuration assumes nodes are running in offline/demo mode
- For full functionality with actual funds, you need to connect to a cardano-node
- See the official Hydra documentation for production configuration

### Step 3: Start the Hydra Nodes

```bash
# Start all nodes
docker-compose up -d

# Check status
docker-compose ps
```

**Expected Output:**
```
NAME     COMMAND                  SERVICE        STATUS    PORTS
alice    "hydra-node --node-i…"   hydra-node-1   Up        0.0.0.0:4001->4001/tcp, 0.0.0.0:5001->5001/tcp
bob      "hydra-node --node-i…"   hydra-node-2   Up        0.0.0.0:4002->4002/tcp, 0.0.0.0:5002->5002/tcp
carol    "hydra-node --node-i…"   hydra-node-3   Up        0.0.0.0:4003->4003/tcp, 0.0.0.0:5003->5003/tcp
```

### Step 4: Check Node Connectivity

```bash
# Check Alice's logs
docker logs alice --tail 20

# Check if nodes are connected
curl -s http://localhost:4001/snapshot/utxo | jq
```

**Expected Output:**
```json
{
  "headId": null,
  "snapshotNumber": 0,
  "utxo": {}
}
```

---

## Sample Workflow

Now let's go through the complete Hydra Head lifecycle!

### Important Note About Examples

The commands and API calls shown in this workflow are **conceptual and educational**. They demonstrate the workflow and logic of Hydra operations but may not match the exact API specifications of your Hydra version. For production implementations:

- Always refer to the official [Hydra API documentation](https://hydra.family/head-protocol/api-reference/)
- Use proper Cardano transaction formats (CBOR-encoded)
- Test thoroughly on testnets before any mainnet use
- Ensure proper key management and security practices

This exercise is designed to help you understand the concepts and workflow of Hydra Layer 2.

### Workflow Overview

```
1. Initialize Head
2. Commit Funds (from Layer 1)
3. Open Head
4. Perform Off-Chain Transactions
5. Close Head
6. Fanout (return funds to Layer 1)
```

### Step 1: Initialize a Hydra Head

The Init command starts the process of creating a Head:

```bash
# Alice initializes the Head
curl -X POST http://localhost:4001 \
  -H "Content-Type: application/json" \
  -d '{"tag": "Init"}'
```

**Expected Response:**
```json
{
  "tag": "CommandResponse",
  "headId": "1234567890abcdef",
  "seq": 0,
  "timestamp": "2024-01-15T10:00:00Z"
}
```

**What's Happening:**
- A unique Head ID is created
- All parties are notified to commit funds
- The Head is in "Initializing" state

**Note:** The actual Hydra API and response formats may vary. Consult the official Hydra API documentation for the exact endpoints and data structures for your version.

### Step 2: Commit Funds to the Head

Each participant commits ADA from their Layer 1 wallet:

```bash
# Alice commits 100 ADA
curl -X POST http://localhost:4001/commit \
  -H "Content-Type: application/json" \
  -d '{
    "Commit": {
      "utxo": {
        "txId": "abc123...",
        "index": 0,
        "value": {
          "lovelace": 100000000
        }
      }
    }
  }'

# Bob commits 50 ADA
curl -X POST http://localhost:4002/commit \
  -H "Content-Type: application/json" \
  -d '{
    "Commit": {
      "utxo": {
        "txId": "def456...",
        "index": 0,
        "value": {
          "lovelace": 50000000
        }
      }
    }
  }'

# Carol commits 75 ADA
curl -X POST http://localhost:4003/commit \
  -H "Content-Type: application/json" \
  -d '{
    "Commit": {
      "utxo": {
        "txId": "ghi789...",
        "index": 0,
        "value": {
          "lovelace": 75000000
        }
      }
    }
  }'
```

**Expected Output (per commit):**
```json
{
  "tag": "Committed",
  "headId": "1234567890abcdef",
  "party": "alice",
  "utxo": {...}
}
```

### Step 3: Open the Head

Once all parties have committed, the Head opens automatically:

```bash
# Check Head status
curl -s http://localhost:4001/snapshot/utxo | jq
```

**Expected Output:**
```json
{
  "headId": "1234567890abcdef",
  "snapshotNumber": 0,
  "utxo": {
    "alice": 100000000,
    "bob": 50000000,
    "carol": 75000000
  },
  "confirmedTransactions": []
}
```

**What's Happening:**
- Head is now OPEN
- Total: 225 ADA locked in the Head
- Participants can now transact off-chain at high speed!

### Step 4: Perform Fast Off-Chain Transactions

Now the exciting part - rapid transactions within the Head:

**Note:** The transaction format shown here is conceptual. Hydra uses Cardano's UTxO model which requires proper transaction construction with inputs, outputs, and witnesses. For actual implementation, use the Hydra API with properly formatted Cardano transactions.

**Conceptual Example:**
```bash
# Alice creates and submits a transaction to Bob
# In practice, you would construct a proper Cardano transaction
curl -X POST http://localhost:4001 \
  -H "Content-Type: application/json" \
  -d '{
    "tag": "NewTx",
    "transaction": {
      "cborHex": "<properly_encoded_cardano_transaction>"
    }
  }'
```

**Expected Output:**
```json
{
  "tag": "TxValid",
  "transactionId": "tx001",
  "timestamp": "2024-01-15T10:05:00.123Z"
}
```

**Transaction Confirmation Time:** ~10-100ms (compared to 20+ seconds on Layer 1!)

**What This Demonstrates:**
- Transactions within a Head are nearly instantaneous
- Multiple transactions can be processed per second
- All participants see the same state updates
- Transactions use standard Cardano transaction format

**For Educational Purposes:**
During your meetup, participants can simulate transactions by:
1. Using the Hydra TUI (Terminal User Interface) tool
2. Building simple transactions with `cardano-cli` and submitting via Hydra API
3. Using example scripts provided in the Hydra repository

```bash
# Example: Check balances after transactions
curl -s http://localhost:4001/snapshot/utxo | jq
```

**Conceptual Result (after multiple transactions):**
```json
{
  "headId": "1234567890abcdef",
  "snapshotNumber": 3,
  "utxo": {
    "<alice_utxo_ref>": {"lovelace": 105000000},
    "<bob_utxo_ref>": {"lovelace": 55000000},
    "<carol_utxo_ref>": {"lovelace": 65000000}
  },
  "confirmedTransactions": ["tx001", "tx002", "tx003"]
}
```

### Step 5: Close the Hydra Head

When participants are done transacting, any party can initiate closing:

```bash
# Alice closes the Head
curl -X POST http://localhost:4001/close \
  -H "Content-Type: application/json" \
  -d '{"Close": {}}'
```

**Expected Output:**
```json
{
  "tag": "HeadIsClosed",
  "headId": "1234567890abcdef",
  "snapshotNumber": 3,
  "contestationDeadline": "2024-01-15T11:00:00Z"
}
```

**What's Happening:**
- Head enters "Closed" state
- A contestation period begins (default: ~1 hour)
- Other parties can contest if they have a more recent snapshot
- After contestation period, funds can be withdrawn

### Step 6: Fanout - Distribute Final Balances

After the contestation period expires:

```bash
# Fanout the final state to Layer 1
curl -X POST http://localhost:4001/fanout \
  -H "Content-Type: application/json" \
  -d '{"Fanout": {}}'
```

**Expected Output:**
```json
{
  "tag": "HeadIsFinalized",
  "headId": "1234567890abcdef",
  "utxo": {
    "alice": 105000000,
    "bob": 55000000,
    "carol": 65000000
  }
}
```

**What's Happening:**
- Final balances are written to Cardano Layer 1
- Each participant receives their final balance
- Head is now closed and can't be reopened

### Step 7: Verify Final Balances on Layer 1

```bash
# Check Alice's wallet on Layer 1
cardano-cli query utxo \
  --address $(cat alice-wallet.addr) \
  --testnet-magic 1

# You should see the final 105 ADA in Alice's wallet
```

---

## Group Activity

### Exercise: "Hydra Payment Race"

**Objective:** Experience the speed and throughput of Hydra by simulating rapid micropayments.

**Setup:**
- Form groups of 3-5 participants
- Each person controls one Hydra node
- Start with equal balances (e.g., 100 ADA each)

**Activity Steps:**

1. **Initialize and Open a Head** (10 minutes)
   - Elect one person to initialize
   - All commit equal amounts
   - Confirm Head is open

2. **Rapid Transaction Phase** (15 minutes)
   - Set a timer for 5 minutes
   - Each person sends random amounts (1-10 ADA) to others
   - Try to make as many transactions as possible
   - Track: How many transactions completed?

3. **Snapshot Comparison** (5 minutes)
   - All participants query their snapshot
   - Compare snapshot numbers - should be identical
   - Verify all balances sum to the original total

4. **Orderly Close** (10 minutes)
   - One person initiates close
   - Wait through contestation period (can be shortened for demo)
   - Execute fanout
   - Verify final balances

### Discussion Questions

After the exercise, discuss:

1. **Speed**: How did Hydra's transaction speed compare to Layer 1?
2. **Cost**: What were the transaction fees within the Head?
3. **Coordination**: What challenges did you face coordinating with others?
4. **Use Cases**: What real-world applications could benefit from this?
5. **Limitations**: What constraints did you notice?

### Competition Element (Optional)

**"Most Transactions in 5 Minutes"**
- Count successful transactions per group
- Prize for the group with highest throughput
- Bonus points for maintaining consensus (matching snapshots)

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Nodes Won't Connect

**Symptoms:**
```
Error: Cannot connect to peer hydra-node-2:5002
```

**Solutions:**
```bash
# Check if containers are running
docker-compose ps

# Check network connectivity
docker exec alice ping hydra-node-2

# Restart the network
docker-compose down
docker-compose up -d

# Check firewall rules
sudo ufw status
```

#### Issue 2: Cannot Commit Funds

**Symptoms:**
```json
{
  "error": "Insufficient funds in wallet"
}
```

**Solutions:**
- Ensure your Layer 1 wallet has sufficient ADA
- Verify wallet is connected to the correct network
- Check that UTXOs are available:
```bash
cardano-cli query utxo --address $(cat wallet.addr)
```

#### Issue 3: Head Won't Open

**Symptoms:**
- Head stuck in "Initializing" state
- Not all parties committed

**Solutions:**
```bash
# Check all nodes' status
curl http://localhost:4001/snapshot/utxo
curl http://localhost:4002/snapshot/utxo
curl http://localhost:4003/snapshot/utxo

# Verify all parties committed
docker logs alice | grep "Committed"
docker logs bob | grep "Committed"
docker logs carol | grep "Committed"

# If a party missed commitment, they need to commit:
curl -X POST http://localhost:400X/commit ...
```

#### Issue 4: Transaction Rejected

**Symptoms:**
```json
{
  "tag": "TxInvalid",
  "reason": "Insufficient balance"
}
```

**Solutions:**
- Check current balances:
```bash
curl -s http://localhost:4001/snapshot/utxo | jq '.utxo'
```
- Ensure sender has sufficient balance
- Verify transaction amount is correct (in lovelace: 1 ADA = 1,000,000 lovelace)

#### Issue 5: Docker Permission Denied

**Symptoms:**
```
Got permission denied while trying to connect to Docker daemon
```

**Solutions:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Log out and back in, or:
newgrp docker

# Alternatively, use sudo (not recommended for production)
sudo docker-compose up -d
```

### Checking Logs

**View real-time logs:**
```bash
# All nodes
docker-compose logs -f

# Specific node
docker logs alice -f

# Last 50 lines
docker logs alice --tail 50
```

### Resetting the Exercise

To start fresh:
```bash
# Stop all containers
docker-compose down

# Remove state data
rm -rf node*/state/*

# Remove logs
rm -rf node*/logs/*

# Start again
docker-compose up -d
```

---

## Additional Resources

### Official Documentation

- **Hydra Documentation**: https://hydra.family/head-protocol/
- **Hydra GitHub Repository**: https://github.com/input-output-hk/hydra
- **Hydra User Manual**: https://hydra.family/head-protocol/docs/getting-started
- **Cardano Documentation**: https://docs.cardano.org/
- **IOG Blog - Hydra**: https://iohk.io/en/blog/posts/2021/09/17/hydra-cardano-s-solution-for-ultimate-scalability/

### Technical Papers

- **Hydra Head Protocol**: [Research Paper](https://iohk.io/en/research/library/papers/hydrafast-isomorphic-state-channels/)
- **Hydra: Fast Isomorphic State Channels**: Academic paper explaining the protocol
- **Ouroboros Consensus**: Understanding Cardano's base layer

### Video Tutorials

- **Hydra Workshop Series**: IOG's official workshop recordings
- **Hydra Deep Dive**: Technical explanation by Hydra team
- **Setting Up Hydra Nodes**: Step-by-step video guide

### Community Resources

- **Hydra Discord**: Join the community for support
- **Cardano Forum - Hydra Section**: Discussions and Q&A
- **Hydra Explorer**: Tool for visualizing Head states
- **Hydra Dashboard**: Web interface for managing Heads

### Development Resources

- **Hydra API Reference**: Complete API documentation
- **Hydra SDK**: Libraries for integrating Hydra in applications
- **Example DApps**: Sample applications using Hydra
- **Testing Frameworks**: Tools for testing Hydra integrations

### Network Resources

- **Cardano Testnet Faucet**: Get free test ADA
  - Preview Testnet: https://docs.cardano.org/cardano-testnet/tools/faucet
  - Pre-Production Testnet: Similar faucet available

### Advanced Topics

- **Multi-Head Architectures**: Connecting multiple Heads
- **Hydra for Payments**: Building payment channels
- **Hydra for Gaming**: Using Hydra for game state
- **Hydra + Smart Contracts**: Plutus scripts in Hydra Heads

---

## Next Steps

After completing this exercise, you can:

1. **Experiment with More Participants**: Try 5+ node networks
2. **Build a Simple DApp**: Create a web interface for your Hydra Head
3. **Integrate with Plutus**: Add smart contract logic to your Heads
4. **Performance Testing**: Measure actual TPS in your environment
5. **Explore Advanced Features**:
   - Incremental commits/decommits
   - Multi-signature schemes
   - Cross-Head communication

### Suggested Follow-Up Labs

1. **Hydra + DeFi**: Build a simple DEX using Hydra
2. **Hydra Gaming**: Create a turn-based game with Hydra state channels
3. **Payment Channel Network**: Connect multiple Heads for routing
4. **Hydra Monitoring**: Set up Grafana dashboards for your nodes

---

## Feedback and Contributions

We'd love to hear about your experience!

- **Found an issue?** Open an issue on GitHub
- **Have suggestions?** Submit a pull request
- **Want to share your project?** Post in discussions

### Acknowledgments

This exercise was created for the Cardano Meetup series to provide hands-on experience with Hydra Layer 2 technology.

**Special thanks to:**
- Input Output Global (IOG) - Hydra development team
- Cardano Foundation - Educational resources
- Community contributors and testers

---

## License

This educational material is provided under MIT License.

**Happy Building! 🚀**

---

*Last Updated: November 2024*  
*Hydra Version: 0.15.0*  
*Cardano Node Version: 8.0+*
