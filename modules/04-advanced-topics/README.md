# Module 4: Advanced Topics 🎓

Explore advanced Solana development concepts.

## Table of Contents

1. [Cross-Program Invocation (CPI)](#cross-program-invocation-cpi)
2. [Token Program Integration](#token-program-integration)
3. [Program Derived Addresses (PDAs) Deep Dive](#program-derived-addresses-pdas-deep-dive)
4. [Security Best Practices](#security-best-practices)
5. [Optimization Techniques](#optimization-techniques)
6. [Real-World Project Ideas](#real-world-project-ideas)

## Cross-Program Invocation (CPI)

CPIs allow your program to call other programs on Solana.

### Basic CPI Example

```rust
use anchor_lang::prelude::*;
use anchor_lang::system_program::{Transfer, transfer};

#[program]
pub mod cpi_example {
    use super::*;

    pub fn transfer_sol(ctx: Context<TransferSol>, amount: u64) -> Result<()> {
        let cpi_accounts = Transfer {
            from: ctx.accounts.from.to_account_info(),
            to: ctx.accounts.to.to_account_info(),
        };
        
        let cpi_program = ctx.accounts.system_program.to_account_info();
        let cpi_ctx = CpiContext::new(cpi_program, cpi_accounts);
        
        transfer(cpi_ctx, amount)?;
        
        msg!("Transferred {} lamports", amount);
        Ok(())
    }
}

#[derive(Accounts)]
pub struct TransferSol<'info> {
    #[account(mut)]
    pub from: Signer<'info>,
    
    /// CHECK: This is the recipient account
    #[account(mut)]
    pub to: AccountInfo<'info>,
    
    pub system_program: Program<'info, System>,
}
```

### CPI with PDA Signer

When your program needs to sign for a PDA:

```rust
pub fn transfer_from_pda(ctx: Context<TransferFromPda>, amount: u64) -> Result<()> {
    let vault = &ctx.accounts.vault;
    
    // Seeds for PDA signing
    let seeds = &[
        b"vault",
        ctx.accounts.authority.key.as_ref(),
        &[vault.bump],
    ];
    let signer_seeds = &[&seeds[..]];
    
    let cpi_accounts = Transfer {
        from: ctx.accounts.vault_token_account.to_account_info(),
        to: ctx.accounts.user_token_account.to_account_info(),
    };
    
    let cpi_program = ctx.accounts.system_program.to_account_info();
    
    // Use CpiContext::new_with_signer for PDA signing
    let cpi_ctx = CpiContext::new_with_signer(
        cpi_program,
        cpi_accounts,
        signer_seeds,
    );
    
    transfer(cpi_ctx, amount)
}
```

## Token Program Integration

### Creating SPL Tokens with Anchor

```rust
use anchor_lang::prelude::*;
use anchor_spl::token::{self, Mint, Token, TokenAccount, MintTo, Transfer};
use anchor_spl::associated_token::AssociatedToken;

#[program]
pub mod token_example {
    use super::*;

    pub fn create_token(ctx: Context<CreateToken>, decimals: u8) -> Result<()> {
        msg!("Token mint created: {}", ctx.accounts.mint.key());
        Ok(())
    }

    pub fn mint_tokens(ctx: Context<MintTokens>, amount: u64) -> Result<()> {
        let cpi_accounts = MintTo {
            mint: ctx.accounts.mint.to_account_info(),
            to: ctx.accounts.token_account.to_account_info(),
            authority: ctx.accounts.mint_authority.to_account_info(),
        };
        
        let cpi_program = ctx.accounts.token_program.to_account_info();
        let cpi_ctx = CpiContext::new(cpi_program, cpi_accounts);
        
        token::mint_to(cpi_ctx, amount)?;
        
        msg!("Minted {} tokens", amount);
        Ok(())
    }

    pub fn transfer_tokens(ctx: Context<TransferTokens>, amount: u64) -> Result<()> {
        let cpi_accounts = Transfer {
            from: ctx.accounts.from.to_account_info(),
            to: ctx.accounts.to.to_account_info(),
            authority: ctx.accounts.authority.to_account_info(),
        };
        
        let cpi_program = ctx.accounts.token_program.to_account_info();
        let cpi_ctx = CpiContext::new(cpi_program, cpi_accounts);
        
        token::transfer(cpi_ctx, amount)?;
        
        msg!("Transferred {} tokens", amount);
        Ok(())
    }
}

#[derive(Accounts)]
pub struct CreateToken<'info> {
    #[account(
        init,
        payer = payer,
        mint::decimals = 9,
        mint::authority = payer,
    )]
    pub mint: Account<'info, Mint>,
    
    #[account(mut)]
    pub payer: Signer<'info>,
    
    pub system_program: Program<'info, System>,
    pub token_program: Program<'info, Token>,
    pub rent: Sysvar<'info, Rent>,
}

#[derive(Accounts)]
pub struct MintTokens<'info> {
    #[account(mut)]
    pub mint: Account<'info, Mint>,
    
    #[account(
        init_if_needed,
        payer = mint_authority,
        associated_token::mint = mint,
        associated_token::authority = mint_authority,
    )]
    pub token_account: Account<'info, TokenAccount>,
    
    #[account(mut)]
    pub mint_authority: Signer<'info>,
    
    pub token_program: Program<'info, Token>,
    pub associated_token_program: Program<'info, AssociatedToken>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct TransferTokens<'info> {
    #[account(mut)]
    pub from: Account<'info, TokenAccount>,
    
    #[account(mut)]
    pub to: Account<'info, TokenAccount>,
    
    pub authority: Signer<'info>,
    
    pub token_program: Program<'info, Token>,
}
```

## Program Derived Addresses (PDAs) Deep Dive

### Multiple Seeds for Complex Data Structures

```rust
// User profile PDA
let (user_profile_pda, _) = Pubkey::find_program_address(
    &[b"profile", user.key().as_ref()],
    program_id
);

// User's game stats for a specific game
let (game_stats_pda, _) = Pubkey::find_program_address(
    &[
        b"game-stats",
        user.key().as_ref(),
        game_id.as_ref(),
    ],
    program_id
);

// Escrow between two parties
let (escrow_pda, _) = Pubkey::find_program_address(
    &[
        b"escrow",
        party_a.key().as_ref(),
        party_b.key().as_ref(),
        &escrow_id.to_le_bytes(),
    ],
    program_id
);
```

### PDA Account Pattern

```rust
#[account]
#[derive(InitSpace)]
pub struct UserProfile {
    pub authority: Pubkey,      // 32 bytes
    pub username: [u8; 32],     // 32 bytes (fixed string)
    pub level: u32,             // 4 bytes
    pub experience: u64,        // 8 bytes
    pub created_at: i64,        // 8 bytes
    pub bump: u8,               // 1 byte
}

#[derive(Accounts)]
#[instruction(username: [u8; 32])]
pub struct CreateProfile<'info> {
    #[account(
        init,
        payer = authority,
        space = 8 + UserProfile::INIT_SPACE,
        seeds = [b"profile", authority.key().as_ref()],
        bump
    )]
    pub profile: Account<'info, UserProfile>,
    
    #[account(mut)]
    pub authority: Signer<'info>,
    
    pub system_program: Program<'info, System>,
}
```

## Security Best Practices

### 1. Validate All Inputs

```rust
pub fn process_payment(ctx: Context<ProcessPayment>, amount: u64) -> Result<()> {
    // Validate amount
    require!(amount > 0, ErrorCode::InvalidAmount);
    require!(amount <= MAX_PAYMENT, ErrorCode::AmountTooLarge);
    
    // Validate accounts
    require!(
        ctx.accounts.recipient.key() != ctx.accounts.sender.key(),
        ErrorCode::SelfTransfer
    );
    
    // Process...
    Ok(())
}
```

### 2. Use Constraints Properly

```rust
#[derive(Accounts)]
pub struct SecureOperation<'info> {
    #[account(
        mut,
        has_one = authority @ ErrorCode::InvalidAuthority,
        constraint = !vault.is_frozen @ ErrorCode::VaultFrozen,
        constraint = vault.balance >= amount @ ErrorCode::InsufficientFunds,
    )]
    pub vault: Account<'info, Vault>,
    
    pub authority: Signer<'info>,
}
```

### 3. Prevent Reentrancy

```rust
#[account]
pub struct ProcessingState {
    pub is_processing: bool,
    // ...
}

pub fn secure_withdraw(ctx: Context<SecureWithdraw>, amount: u64) -> Result<()> {
    let state = &mut ctx.accounts.state;
    
    // Check reentrancy guard
    require!(!state.is_processing, ErrorCode::ReentrancyGuard);
    
    // Set guard
    state.is_processing = true;
    
    // Perform withdrawal logic...
    
    // Clear guard
    state.is_processing = false;
    
    Ok(())
}
```

### 4. Secure Random Number Generation

```rust
use anchor_lang::solana_program::sysvar::clock::Clock;

pub fn generate_pseudo_random(ctx: Context<RandomContext>) -> Result<u64> {
    let clock = Clock::get()?;
    let slot = clock.slot;
    let timestamp = clock.unix_timestamp;
    
    // Combine multiple sources (NOT cryptographically secure!)
    let seed = slot
        .wrapping_add(timestamp as u64)
        .wrapping_add(ctx.accounts.user.key().to_bytes()[0] as u64);
    
    // For true randomness, use VRF (Verifiable Random Function)
    // See: Switchboard or Chainlink VRF
    
    Ok(seed)
}
```

### 5. Account Ownership Verification

```rust
#[derive(Accounts)]
pub struct VerifiedAccounts<'info> {
    // Verify program ownership
    #[account(
        owner = token::ID @ ErrorCode::InvalidTokenAccount
    )]
    pub token_account: Account<'info, TokenAccount>,
    
    // Verify specific program
    #[account(
        constraint = custom_account.owner == &crate::ID @ ErrorCode::InvalidOwner
    )]
    /// CHECK: We verify ownership manually
    pub custom_account: AccountInfo<'info>,
}
```

## Optimization Techniques

### 1. Minimize Account Size

```rust
// Inefficient
#[account]
pub struct Inefficient {
    pub data: Vec<u8>,        // Dynamic size = expensive
    pub description: String,   // Dynamic size
}

// Efficient
#[account]
pub struct Efficient {
    pub data: [u8; 32],       // Fixed size
    pub description: [u8; 64], // Fixed size
}
```

### 2. Use Zero-Copy for Large Accounts

```rust
use anchor_lang::prelude::*;

#[account(zero_copy)]
#[repr(C)]
pub struct LargeAccount {
    pub authority: Pubkey,
    pub data: [u64; 1000], // Large fixed array
}

#[derive(Accounts)]
pub struct UseLargeAccount<'info> {
    #[account(mut)]
    pub large_account: AccountLoader<'info, LargeAccount>,
}

pub fn update_large_account(ctx: Context<UseLargeAccount>, index: usize, value: u64) -> Result<()> {
    let mut account = ctx.accounts.large_account.load_mut()?;
    account.data[index] = value;
    Ok(())
}
```

### 3. Batch Operations

```rust
pub fn batch_transfer(
    ctx: Context<BatchTransfer>,
    amounts: Vec<u64>,
) -> Result<()> {
    require!(amounts.len() <= MAX_BATCH_SIZE, ErrorCode::BatchTooLarge);
    
    for (i, amount) in amounts.iter().enumerate() {
        // Process each transfer
        // This is more efficient than multiple transactions
    }
    
    Ok(())
}
```

### 4. Compute Budget

```typescript
// Client-side: Request more compute units if needed
import { ComputeBudgetProgram } from "@solana/web3.js";

const modifyComputeUnits = ComputeBudgetProgram.setComputeUnitLimit({
    units: 400_000, // Default is 200,000
});

const transaction = new Transaction()
    .add(modifyComputeUnits)
    .add(yourInstruction);
```

## Real-World Project Ideas

### 1. Decentralized Voting System

Build a voting system with:
- Proposal creation
- Token-weighted voting
- Time-locked execution

```rust
#[account]
pub struct Proposal {
    pub creator: Pubkey,
    pub description: [u8; 256],
    pub yes_votes: u64,
    pub no_votes: u64,
    pub deadline: i64,
    pub executed: bool,
}
```

### 2. NFT Staking Program

Allow users to stake NFTs for rewards:
- Stake/unstake NFTs
- Calculate rewards based on time staked
- Claim accumulated rewards

### 3. Escrow Service

Create a trustless escrow:
- Deposit funds
- Multi-party approval
- Dispute resolution
- Automatic release

### 4. On-Chain Game

Build a simple game:
- Player registration
- Game state management
- Leaderboard
- Rewards distribution

### 5. Token Vesting

Implement token vesting with:
- Linear/cliff vesting
- Multiple beneficiaries
- Revocation capability
- Claim mechanism

## Testing Advanced Programs

### Integration Tests

```typescript
describe("Advanced Integration Tests", () => {
  it("Tests cross-program invocation", async () => {
    // Setup accounts
    // ...
    
    // Execute CPI
    const tx = await program.methods
      .crossProgramCall()
      .accounts({
        // ...
      })
      .remainingAccounts([
        { pubkey: externalProgramId, isSigner: false, isWritable: false },
      ])
      .rpc();
    
    // Verify results
    // ...
  });
  
  it("Tests error handling", async () => {
    try {
      await program.methods
        .riskyOperation()
        .accounts({/* ... */})
        .rpc();
      
      assert.fail("Expected error was not thrown");
    } catch (error) {
      expect(error.error.errorCode.code).to.equal("InsufficientFunds");
    }
  });
});
```

### Fuzzing

```bash
# Install cargo-fuzz
cargo install cargo-fuzz

# Create fuzz target
cargo fuzz init

# Run fuzzer
cargo +nightly fuzz run fuzz_target_1
```

---

## Conclusion

Congratulations on completing the Solana x Rust Tutorial! You've learned:

✅ Rust fundamentals for blockchain development  
✅ Solana's account model and architecture  
✅ Building and deploying Solana programs  
✅ Advanced patterns like CPI and token integration  
✅ Security best practices  
✅ Optimization techniques  

## What's Next?

1. **Build Projects**: Apply what you've learned by building real projects
2. **Join Communities**: Engage with the Solana developer community
3. **Contribute**: Contribute to open-source Solana projects
4. **Stay Updated**: Solana evolves rapidly; keep learning!

## Additional Resources

- [Solana Program Library (SPL)](https://spl.solana.com/)
- [Metaplex](https://www.metaplex.com/) - NFT infrastructure
- [Marinade Finance](https://marinade.finance/) - Liquid staking
- [Orca](https://www.orca.so/) - DEX example
- [Auditing Resources](https://github.com/slowmist/solana-security-best-practices)

Happy building! 🚀
