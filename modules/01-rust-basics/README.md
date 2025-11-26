# Module 1: Rust Basics 🦀

Learn the fundamentals of Rust programming language essential for Solana development.

## Table of Contents

1. [Why Rust for Solana?](#why-rust-for-solana)
2. [Variables and Data Types](#variables-and-data-types)
3. [Functions](#functions)
4. [Ownership and Borrowing](#ownership-and-borrowing)
5. [Structs and Enums](#structs-and-enums)
6. [Error Handling](#error-handling)
7. [Exercises](#exercises)

## Why Rust for Solana?

Solana chose Rust as its primary programming language for several reasons:

- **Memory Safety**: Rust prevents memory-related bugs at compile time
- **Performance**: Zero-cost abstractions and no garbage collector
- **Concurrency**: Safe concurrent programming
- **Modern Tooling**: Excellent package manager (Cargo) and documentation

## Variables and Data Types

### Immutability by Default

```rust
fn main() {
    let x = 5; // Immutable by default
    // x = 6; // This would cause a compile error!
    
    let mut y = 5; // Mutable variable
    y = 6; // This is allowed
    
    println!("x = {}, y = {}", x, y);
}
```

### Common Data Types

```rust
fn main() {
    // Integers
    let a: i32 = 42;        // Signed 32-bit integer
    let b: u64 = 100;       // Unsigned 64-bit integer
    
    // Floating point
    let c: f64 = 3.14;
    
    // Boolean
    let is_active: bool = true;
    
    // Characters and Strings
    let letter: char = 'A';
    let greeting: &str = "Hello, Solana!";
    let owned_string: String = String::from("Hello");
    
    // Arrays and Vectors
    let arr: [i32; 3] = [1, 2, 3];           // Fixed-size array
    let vec: Vec<i32> = vec![1, 2, 3, 4];    // Dynamic vector
}
```

## Functions

### Basic Functions

```rust
fn main() {
    let result = add(5, 3);
    println!("5 + 3 = {}", result);
}

fn add(a: i32, b: i32) -> i32 {
    a + b // No semicolon = return value
}
```

### Functions with Multiple Returns

```rust
fn divide(a: i32, b: i32) -> (i32, i32) {
    let quotient = a / b;
    let remainder = a % b;
    (quotient, remainder)
}

fn main() {
    let (q, r) = divide(10, 3);
    println!("10 / 3 = {} remainder {}", q, r);
}
```

## Ownership and Borrowing

This is Rust's most unique feature and critical for Solana development.

### Ownership Rules

1. Each value has an owner
2. There can only be one owner at a time
3. When the owner goes out of scope, the value is dropped

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1; // s1 is moved to s2
    // println!("{}", s1); // Error! s1 is no longer valid
    println!("{}", s2); // Works!
}
```

### Borrowing

```rust
fn main() {
    let s1 = String::from("hello");
    
    // Immutable borrow
    let len = calculate_length(&s1);
    println!("Length of '{}' is {}", s1, len); // s1 is still valid!
    
    // Mutable borrow
    let mut s2 = String::from("hello");
    change_string(&mut s2);
    println!("{}", s2); // Prints "hello, world"
}

fn calculate_length(s: &String) -> usize {
    s.len()
}

fn change_string(s: &mut String) {
    s.push_str(", world");
}
```

## Structs and Enums

### Structs

```rust
// Define a struct
struct Wallet {
    address: String,
    balance: u64,
    is_active: bool,
}

impl Wallet {
    // Associated function (constructor)
    fn new(address: String) -> Self {
        Wallet {
            address,
            balance: 0,
            is_active: true,
        }
    }
    
    // Method
    fn deposit(&mut self, amount: u64) {
        self.balance += amount;
    }
    
    fn get_balance(&self) -> u64 {
        self.balance
    }
}

fn main() {
    let mut wallet = Wallet::new(String::from("ABC123"));
    wallet.deposit(100);
    println!("Balance: {} SOL", wallet.get_balance());
}
```

### Enums

```rust
enum TransactionType {
    Transfer { amount: u64, recipient: String },
    Stake { amount: u64, validator: String },
    Vote { proposal_id: u32, approve: bool },
}

fn process_transaction(tx: TransactionType) {
    match tx {
        TransactionType::Transfer { amount, recipient } => {
            println!("Transferring {} to {}", amount, recipient);
        }
        TransactionType::Stake { amount, validator } => {
            println!("Staking {} with {}", amount, validator);
        }
        TransactionType::Vote { proposal_id, approve } => {
            let vote = if approve { "yes" } else { "no" };
            println!("Voting {} on proposal {}", vote, proposal_id);
        }
    }
}
```

## Error Handling

### Result Type

```rust
use std::fs::File;
use std::io::Read;

fn read_file_contents(path: &str) -> Result<String, std::io::Error> {
    let mut file = File::open(path)?; // ? propagates errors
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

fn main() {
    match read_file_contents("config.txt") {
        Ok(contents) => println!("File contents: {}", contents),
        Err(e) => println!("Error reading file: {}", e),
    }
}
```

### Custom Errors (Common in Solana)

```rust
use std::error::Error;
use std::fmt;

#[derive(Debug)]
enum WalletError {
    InsufficientFunds,
    InvalidAddress,
    TransactionFailed(String),
}

impl fmt::Display for WalletError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            WalletError::InsufficientFunds => write!(f, "Insufficient funds"),
            WalletError::InvalidAddress => write!(f, "Invalid wallet address"),
            WalletError::TransactionFailed(msg) => write!(f, "Transaction failed: {}", msg),
        }
    }
}

impl Error for WalletError {}

fn transfer(amount: u64, balance: u64) -> Result<u64, WalletError> {
    if amount > balance {
        return Err(WalletError::InsufficientFunds);
    }
    Ok(balance - amount)
}
```

## Exercises

### Exercise 1: Token Balance Tracker

Create a program that tracks token balances for multiple addresses.

```rust
// TODO: Implement a TokenTracker struct with methods to:
// 1. Add a new address with initial balance
// 2. Transfer tokens between addresses
// 3. Get balance for an address
// 4. Print all balances
```

### Exercise 2: Transaction History

Implement a transaction history system using enums and vectors.

```rust
// TODO: Create an enum for different transaction types
// and a struct to store transaction history
```

### Exercise 3: Error Handling Practice

Create a function that validates a Solana-like address format and returns appropriate errors.

```rust
// TODO: Implement address validation with proper error handling
// A valid address should be:
// - Exactly 44 characters long
// - Contain only base58 characters
```

---

## Next Steps

Once you're comfortable with these Rust basics, move on to [Module 2: Solana Fundamentals](../02-solana-fundamentals/README.md).

## Additional Resources

- [Rust Book](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Rustlings](https://github.com/rust-lang/rustlings) - Small exercises to learn Rust
