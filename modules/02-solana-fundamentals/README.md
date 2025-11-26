# Module 2: Solana Fundamentals ☀️

Understand the core concepts of Solana blockchain.

## Table of Contents

1. [What is Solana?](#what-is-solana)
2. [Key Concepts](#key-concepts)
3. [Accounts Model](#accounts-model)
4. [Programs (Smart Contracts)](#programs-smart-contracts)
5. [Transactions](#transactions)
6. [Solana CLI Basics](#solana-cli-basics)
7. [Exercises](#exercises)

## What is Solana?

Solana is a high-performance blockchain designed for decentralized applications and crypto-currencies. Key features include:

- **High Throughput**: ~65,000 transactions per second
- **Low Latency**: ~400ms block times
- **Low Fees**: Fractions of a cent per transaction
- **Proof of History (PoH)**: Novel consensus mechanism

### How Solana Differs from Ethereum

| Feature | Solana | Ethereum |
|---------|--------|----------|
| TPS | ~65,000 | ~15-30 |
| Block Time | 400ms | ~12 seconds |
| Transaction Fee | ~$0.00025 | $1-50+ |
| Smart Contract Language | Rust, C | Solidity |
| Account Model | Account-based | Account-based |
| State Storage | Separate from programs | Combined |

## Key Concepts

### 1. Clusters

Solana has different network environments:

- **Mainnet Beta**: Production network with real SOL
- **Devnet**: Development network for testing (free SOL)
- **Testnet**: Network for stress testing
- **Localnet**: Local development cluster

```bash
# Switch between clusters
solana config set --url mainnet-beta
solana config set --url devnet
solana config set --url localhost
```

### 2. Lamports

The smallest unit of SOL (Solana's native token):

- 1 SOL = 1,000,000,000 lamports (10^9)
- Similar to wei in Ethereum

```rust
// In Solana programs, you work with lamports
const LAMPORTS_PER_SOL: u64 = 1_000_000_000;

let amount_sol: f64 = 1.5;
let amount_lamports: u64 = (amount_sol * LAMPORTS_PER_SOL as f64) as u64;
```

### 3. Keypairs and Addresses

```bash
# Generate a new keypair
solana-keygen new --outfile ~/my-wallet.json

# Get your public key (address)
solana address

# View keypair info
solana-keygen pubkey ~/my-wallet.json
```

## Accounts Model

In Solana, **everything is an account**. This is crucial to understand!

### Account Structure

```rust
pub struct Account {
    /// Lamports in the account
    pub lamports: u64,
    /// Data held in this account
    pub data: Vec<u8>,
    /// The program that owns this account
    pub owner: Pubkey,
    /// Is this account executable?
    pub executable: bool,
    /// The next epoch this account will owe rent
    pub rent_epoch: u64,
}
```

### Types of Accounts

1. **System Accounts**: Regular user wallets
2. **Program Accounts**: Deployed programs (executable)
3. **Data Accounts**: Store program state (owned by programs)
4. **Token Accounts**: Store SPL token balances

### Account Ownership

```
┌─────────────────────────────────────────────────────────────┐
│                    System Program                            │
│            (owns user wallet accounts)                       │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ owns
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    User Wallet Account                       │
│         Address: ABC123...                                   │
│         Balance: 10 SOL                                      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    Your Program                              │
│         (deployed and executable)                            │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ owns
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Program Data Account                      │
│         Stores your program's state                          │
│         (e.g., counter value, user data)                     │
└─────────────────────────────────────────────────────────────┘
```

### Rent

Accounts must maintain a minimum balance (rent-exempt threshold) to stay alive:

```bash
# Check rent-exempt minimum for an account size
solana rent 100  # For 100 bytes of data
```

## Programs (Smart Contracts)

Solana programs are stateless and read-only. They process instructions and modify account data.

### Program Structure

```
┌─────────────────────────────────────────────────────────────┐
│                        Transaction                           │
├─────────────────────────────────────────────────────────────┤
│  Instruction 1                                               │
│  ├── Program ID: TokenProgram                                │
│  ├── Accounts: [sender, receiver, token_account]             │
│  └── Data: [transfer_amount]                                 │
├─────────────────────────────────────────────────────────────┤
│  Instruction 2                                               │
│  ├── Program ID: YourProgram                                 │
│  ├── Accounts: [user, data_account]                          │
│  └── Data: [custom_data]                                     │
└─────────────────────────────────────────────────────────────┘
```

### Entry Point

Every Solana program has an entry point:

```rust
use solana_program::{
    account_info::AccountInfo,
    entrypoint,
    entrypoint::ProgramResult,
    pubkey::Pubkey,
};

// Declare the program entry point
entrypoint!(process_instruction);

pub fn process_instruction(
    program_id: &Pubkey,        // Your program's address
    accounts: &[AccountInfo],    // Accounts passed to the instruction
    instruction_data: &[u8],     // Custom data for the instruction
) -> ProgramResult {
    // Your program logic here
    Ok(())
}
```

## Transactions

### Transaction Structure

```rust
pub struct Transaction {
    /// Signatures for the transaction
    pub signatures: Vec<Signature>,
    /// The message to sign
    pub message: Message,
}

pub struct Message {
    /// The message header
    pub header: MessageHeader,
    /// All account addresses used in this transaction
    pub account_keys: Vec<Pubkey>,
    /// Recent blockhash for transaction validity
    pub recent_blockhash: Hash,
    /// Instructions to execute
    pub instructions: Vec<CompiledInstruction>,
}
```

### Transaction Lifecycle

```
1. Create Transaction
        │
        ▼
2. Sign Transaction
        │
        ▼
3. Send to RPC Node
        │
        ▼
4. Validate & Forward to Leader
        │
        ▼
5. Execute Instructions
        │
        ▼
6. Confirm & Propagate
```

## Solana CLI Basics

### Configuration

```bash
# View current config
solana config get

# Set cluster
solana config set --url devnet

# Set keypair
solana config set --keypair ~/my-wallet.json
```

### Account Operations

```bash
# Check balance
solana balance

# Check another address's balance
solana balance <ADDRESS>

# Airdrop SOL (devnet only)
solana airdrop 2

# Transfer SOL
solana transfer <RECIPIENT_ADDRESS> 1 --allow-unfunded-recipient
```

### Program Operations

```bash
# Deploy a program
solana program deploy target/deploy/my_program.so

# Show program info
solana program show <PROGRAM_ID>

# Close a program (recover SOL)
solana program close <PROGRAM_ID>
```

### Transaction History

```bash
# View recent transactions
solana transaction-history <ADDRESS>

# Get transaction details
solana confirm <TRANSACTION_SIGNATURE> -v
```

## Exercises

### Exercise 1: Explore Devnet

```bash
# 1. Configure Solana CLI for devnet
solana config set --url devnet

# 2. Generate a new keypair
solana-keygen new --outfile ~/devnet-wallet.json

# 3. Set it as default
solana config set --keypair ~/devnet-wallet.json

# 4. Get some SOL
solana airdrop 2

# 5. Check your balance
solana balance
```

### Exercise 2: Account Exploration

Use the Solana Explorer to explore accounts:

1. Go to [Solana Explorer](https://explorer.solana.com/?cluster=devnet)
2. Search for your wallet address
3. Find the System Program: `11111111111111111111111111111111`
4. Find the Token Program: `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA`

### Exercise 3: Understanding PDAs (Program Derived Addresses)

PDAs are special addresses derived from a program ID and seeds:

```rust
use solana_program::pubkey::Pubkey;

// Find a PDA
let (pda, bump) = Pubkey::find_program_address(
    &[
        b"user-stats",           // seed 1: string literal
        user_pubkey.as_ref(),    // seed 2: user's public key
    ],
    program_id,
);
```

**Why PDAs?**
- Programs can "sign" for PDAs they own
- Deterministic addresses based on seeds
- Great for storing user-specific data

---

## Key Takeaways

1. **Everything is an account** - wallets, programs, and data
2. **Programs are stateless** - they modify account data
3. **Accounts have owners** - only the owner can modify data
4. **PDAs** allow programs to have authority over accounts
5. **Rent** keeps the network clean of unused accounts

## Next Steps

Now that you understand Solana fundamentals, let's build your first program in [Module 3: Building Your First Solana Program](../03-first-program/README.md).

## Additional Resources

- [Solana Docs - Core Concepts](https://docs.solana.com/developing/programming-model/overview)
- [Solana Cookbook](https://solanacookbook.com/)
- [Solana Bytes (YouTube)](https://www.youtube.com/@SolanaFndn)
