# Module 3: Building Your First Solana Program 🔨

Create and deploy your first Solana program.

## Table of Contents

1. [Project Setup](#project-setup)
2. [Understanding the Structure](#understanding-the-structure)
3. [Building a Counter Program](#building-a-counter-program)
4. [Testing Your Program](#testing-your-program)
5. [Deploying to Devnet](#deploying-to-devnet)
6. [Interacting with Your Program](#interacting-with-your-program)

## Project Setup

We'll use the **Anchor framework** to build our program. Anchor provides a higher-level abstraction that makes Solana development easier.

### Create a New Anchor Project

```bash
# Create a new Anchor project
anchor init counter
cd counter

# Project structure
counter/
├── Anchor.toml          # Project configuration
├── Cargo.toml           # Rust dependencies
├── programs/
│   └── counter/
│       ├── Cargo.toml
│       └── src/
│           └── lib.rs   # Your program code
├── tests/
│   └── counter.ts       # TypeScript tests
├── migrations/
│   └── deploy.ts
└── app/                 # Frontend (optional)
```

### Configure for Devnet

Edit `Anchor.toml`:

```toml
[features]
seeds = false
skip-lint = false

[programs.devnet]
counter = "YOUR_PROGRAM_ID_WILL_GO_HERE"

[registry]
url = "https://api.apr.dev"

[provider]
cluster = "devnet"
wallet = "~/.config/solana/id.json"

[scripts]
test = "yarn run ts-mocha -p ./tsconfig.json -t 1000000 tests/**/*.ts"
```

## Understanding the Structure

### Anchor Program Anatomy

```rust
use anchor_lang::prelude::*;

// Program ID - unique identifier for your program
declare_id!("YOUR_PROGRAM_ID");

// Program module - contains all instructions
#[program]
pub mod counter {
    use super::*;

    // Instructions (functions users can call)
    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        // Logic here
        Ok(())
    }
}

// Account structures with validation
#[derive(Accounts)]
pub struct Initialize<'info> {
    // Account definitions
}

// Data structures stored in accounts
#[account]
pub struct Counter {
    pub count: u64,
}
```

## Building a Counter Program

Let's build a simple counter that can:
1. Initialize a new counter
2. Increment the counter
3. Decrement the counter

### Step 1: Define the Program

Edit `programs/counter/src/lib.rs`:

```rust
use anchor_lang::prelude::*;

declare_id!("11111111111111111111111111111111"); // Will be updated after build

#[program]
pub mod counter {
    use super::*;

    /// Initialize a new counter account
    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        let counter = &mut ctx.accounts.counter;
        counter.count = 0;
        counter.authority = ctx.accounts.authority.key();
        counter.bump = ctx.bumps.counter;
        
        msg!("Counter initialized! Current count: {}", counter.count);
        Ok(())
    }

    /// Increment the counter by 1
    pub fn increment(ctx: Context<Update>) -> Result<()> {
        let counter = &mut ctx.accounts.counter;
        counter.count = counter.count.checked_add(1)
            .ok_or(ErrorCode::Overflow)?;
        
        msg!("Counter incremented! Current count: {}", counter.count);
        Ok(())
    }

    /// Decrement the counter by 1
    pub fn decrement(ctx: Context<Update>) -> Result<()> {
        let counter = &mut ctx.accounts.counter;
        counter.count = counter.count.checked_sub(1)
            .ok_or(ErrorCode::Underflow)?;
        
        msg!("Counter decremented! Current count: {}", counter.count);
        Ok(())
    }

    /// Set the counter to a specific value
    pub fn set(ctx: Context<Update>, value: u64) -> Result<()> {
        let counter = &mut ctx.accounts.counter;
        counter.count = value;
        
        msg!("Counter set to: {}", counter.count);
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(
        init,
        payer = authority,
        space = 8 + Counter::INIT_SPACE,
        seeds = [b"counter", authority.key().as_ref()],
        bump
    )]
    pub counter: Account<'info, Counter>,
    
    #[account(mut)]
    pub authority: Signer<'info>,
    
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct Update<'info> {
    #[account(
        mut,
        seeds = [b"counter", authority.key().as_ref()],
        bump = counter.bump,
        has_one = authority
    )]
    pub counter: Account<'info, Counter>,
    
    pub authority: Signer<'info>,
}

#[account]
#[derive(InitSpace)]
pub struct Counter {
    pub count: u64,
    pub authority: Pubkey,
    pub bump: u8,
}

#[error_code]
pub enum ErrorCode {
    #[msg("Counter overflow")]
    Overflow,
    #[msg("Counter underflow")]
    Underflow,
}
```

### Step 2: Build the Program

```bash
# Build the program
anchor build

# Get the program ID
solana address -k target/deploy/counter-keypair.json

# Update declare_id! in lib.rs with the output above
# Then rebuild
anchor build
```

### Step 3: Update Program ID

After building, update `declare_id!` with your actual program ID:

```rust
declare_id!("YourActualProgramIdHere123456789");
```

Also update `Anchor.toml`:

```toml
[programs.devnet]
counter = "YourActualProgramIdHere123456789"
```

## Testing Your Program

### TypeScript Tests

Edit `tests/counter.ts`:

```typescript
import * as anchor from "@coral-xyz/anchor";
import { Program } from "@coral-xyz/anchor";
import { Counter } from "../target/types/counter";
import { expect } from "chai";

describe("counter", () => {
  // Configure the client to use the local cluster
  const provider = anchor.AnchorProvider.env();
  anchor.setProvider(provider);

  const program = anchor.workspace.Counter as Program<Counter>;
  const authority = provider.wallet.publicKey;

  // Derive the PDA for the counter
  const [counterPDA] = anchor.web3.PublicKey.findProgramAddressSync(
    [Buffer.from("counter"), authority.toBuffer()],
    program.programId
  );

  it("Initializes the counter", async () => {
    const tx = await program.methods
      .initialize()
      .accounts({
        counter: counterPDA,
        authority: authority,
        systemProgram: anchor.web3.SystemProgram.programId,
      })
      .rpc();

    console.log("Initialize transaction signature:", tx);

    // Fetch the counter account
    const counterAccount = await program.account.counter.fetch(counterPDA);
    expect(counterAccount.count.toNumber()).to.equal(0);
    expect(counterAccount.authority.toString()).to.equal(authority.toString());
  });

  it("Increments the counter", async () => {
    await program.methods
      .increment()
      .accounts({
        counter: counterPDA,
        authority: authority,
      })
      .rpc();

    const counterAccount = await program.account.counter.fetch(counterPDA);
    expect(counterAccount.count.toNumber()).to.equal(1);
  });

  it("Increments the counter again", async () => {
    await program.methods
      .increment()
      .accounts({
        counter: counterPDA,
        authority: authority,
      })
      .rpc();

    const counterAccount = await program.account.counter.fetch(counterPDA);
    expect(counterAccount.count.toNumber()).to.equal(2);
  });

  it("Decrements the counter", async () => {
    await program.methods
      .decrement()
      .accounts({
        counter: counterPDA,
        authority: authority,
      })
      .rpc();

    const counterAccount = await program.account.counter.fetch(counterPDA);
    expect(counterAccount.count.toNumber()).to.equal(1);
  });

  it("Sets the counter to a specific value", async () => {
    await program.methods
      .set(new anchor.BN(100))
      .accounts({
        counter: counterPDA,
        authority: authority,
      })
      .rpc();

    const counterAccount = await program.account.counter.fetch(counterPDA);
    expect(counterAccount.count.toNumber()).to.equal(100);
  });
});
```

### Run Tests

```bash
# Start local validator in a separate terminal
solana-test-validator

# Run tests
anchor test --skip-local-validator

# Or run tests with local validator
anchor test
```

## Deploying to Devnet

### Step 1: Configure Wallet

```bash
# Make sure you have SOL on devnet
solana config set --url devnet
solana airdrop 2
solana balance
```

### Step 2: Deploy

```bash
# Deploy to devnet
anchor deploy --provider.cluster devnet

# Or use
anchor deploy
```

### Step 3: Verify Deployment

```bash
# Check program deployment
solana program show <YOUR_PROGRAM_ID>

# View on explorer
# https://explorer.solana.com/address/<YOUR_PROGRAM_ID>?cluster=devnet
```

## Interacting with Your Program

### Using the Anchor Client

Create a script `scripts/interact.ts`:

```typescript
import * as anchor from "@coral-xyz/anchor";
import { Program } from "@coral-xyz/anchor";
import { Counter } from "../target/types/counter";

async function main() {
  // Configure the client
  const provider = anchor.AnchorProvider.env();
  anchor.setProvider(provider);

  const program = anchor.workspace.Counter as Program<Counter>;
  const authority = provider.wallet.publicKey;

  // Derive PDA
  const [counterPDA] = anchor.web3.PublicKey.findProgramAddressSync(
    [Buffer.from("counter"), authority.toBuffer()],
    program.programId
  );

  console.log("Program ID:", program.programId.toString());
  console.log("Authority:", authority.toString());
  console.log("Counter PDA:", counterPDA.toString());

  // Check if counter exists
  try {
    const counterAccount = await program.account.counter.fetch(counterPDA);
    console.log("Current count:", counterAccount.count.toNumber());
  } catch (e) {
    console.log("Counter not initialized. Initializing...");
    await program.methods
      .initialize()
      .accounts({
        counter: counterPDA,
        authority: authority,
        systemProgram: anchor.web3.SystemProgram.programId,
      })
      .rpc();
    console.log("Counter initialized!");
  }

  // Increment
  console.log("\nIncrementing counter...");
  await program.methods
    .increment()
    .accounts({
      counter: counterPDA,
      authority: authority,
    })
    .rpc();

  // Fetch updated value
  const updated = await program.account.counter.fetch(counterPDA);
  console.log("Updated count:", updated.count.toNumber());
}

main().catch(console.error);
```

Run it:

```bash
npx ts-node scripts/interact.ts
```

### Using JavaScript/Frontend

```javascript
import { Connection, PublicKey } from "@solana/web3.js";
import { AnchorProvider, Program } from "@coral-xyz/anchor";
import { IDL } from "./counter"; // Generated IDL

// Connect to cluster
const connection = new Connection("https://api.devnet.solana.com");

// Your program ID
const programId = new PublicKey("YOUR_PROGRAM_ID");

// Create provider with wallet
const provider = new AnchorProvider(connection, wallet, {});
const program = new Program(IDL, programId, provider);

// Interact with program
async function incrementCounter() {
  const authority = provider.wallet.publicKey;
  
  const [counterPDA] = PublicKey.findProgramAddressSync(
    [Buffer.from("counter"), authority.toBuffer()],
    programId
  );

  await program.methods
    .increment()
    .accounts({
      counter: counterPDA,
      authority: authority,
    })
    .rpc();
}
```

## Common Patterns

### 1. Account Initialization

```rust
#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(
        init,                          // Create new account
        payer = payer,                 // Who pays for rent
        space = 8 + MyData::INIT_SPACE // Account size
    )]
    pub my_account: Account<'info, MyData>,
    
    #[account(mut)]
    pub payer: Signer<'info>,
    
    pub system_program: Program<'info, System>,
}
```

### 2. PDA Seeds

```rust
#[account(
    seeds = [b"user-data", user.key().as_ref()],
    bump
)]
pub user_data: Account<'info, UserData>,
```

### 3. Access Control

```rust
#[account(
    mut,
    has_one = authority,  // Verify authority matches
    constraint = !data.is_locked @ ErrorCode::Locked
)]
pub data: Account<'info, MyData>,

pub authority: Signer<'info>,
```

## Exercises

### Exercise 1: Add a Reset Function

Add a `reset` instruction that sets the counter back to 0.

### Exercise 2: Add Multiple Counters

Modify the program to allow users to create multiple named counters.

### Exercise 3: Add Events

Emit events when the counter changes:

```rust
#[event]
pub struct CounterChanged {
    pub old_value: u64,
    pub new_value: u64,
    pub authority: Pubkey,
}
```

---

## Next Steps

Congratulations! You've built and deployed your first Solana program. Continue to [Module 4: Advanced Topics](../04-advanced-topics/README.md) to learn about more complex concepts.

## Troubleshooting

### Common Errors

1. **"Account not found"**: The account hasn't been initialized yet
2. **"Constraint violation"**: Account validation failed
3. **"Insufficient funds"**: Need more SOL for rent/fees
4. **"Program failed to complete"**: Check your program logic

### Debugging Tips

```bash
# View program logs
solana logs <PROGRAM_ID>

# Get transaction details
solana confirm <TX_SIGNATURE> -v
```
