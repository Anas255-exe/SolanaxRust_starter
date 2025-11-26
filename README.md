# Solana x Rust Starter Tutorial 🚀

A comprehensive, fast-paced tutorial to learn Solana blockchain development with Rust programming.

## 📚 Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Setup](#setup)
4. [Tutorial Modules](#tutorial-modules)
5. [Resources](#resources)

## Introduction

Welcome to the Solana x Rust Starter Tutorial! This repository is designed to take you from zero to building decentralized applications (dApps) on the Solana blockchain using Rust.

### What You'll Learn

- **Rust Fundamentals**: Core concepts of the Rust programming language
- **Solana Basics**: Understanding the Solana blockchain architecture
- **Smart Contracts**: Building programs (smart contracts) on Solana
- **Client Integration**: Connecting frontend applications to Solana programs

## Prerequisites

Before starting this tutorial, you should have:

- Basic programming knowledge (any language)
- Familiarity with command-line interfaces
- A computer running macOS, Linux, or Windows (with WSL2)

## Setup

### 1. Install Rust

```bash
# Install Rust using rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Verify installation
rustc --version
cargo --version
```

### 2. Install Solana CLI Tools

```bash
# Install Solana CLI
sh -c "$(curl -sSfL https://release.anza.xyz/stable/install)"

# Add Solana to PATH (add to your shell profile)
export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"

# Verify installation
solana --version
```

### 3. Install Anchor Framework (Recommended)

```bash
# Install Anchor Version Manager (avm)
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force

# Install latest Anchor
avm install latest
avm use latest

# Verify installation
anchor --version
```

### 4. Configure Solana for Development

```bash
# Set to devnet for development
solana config set --url devnet

# Generate a new keypair for development
solana-keygen new

# Airdrop some SOL for testing
solana airdrop 2
```

## Tutorial Modules

### [Module 1: Rust Basics](modules/01-rust-basics/README.md)
Learn the fundamentals of Rust programming language essential for Solana development.

### [Module 2: Solana Fundamentals](modules/02-solana-fundamentals/README.md)
Understand the core concepts of Solana blockchain.

### [Module 3: Building Your First Solana Program](modules/03-first-program/README.md)
Create and deploy your first Solana program.

### [Module 4: Advanced Topics](modules/04-advanced-topics/README.md)
Explore advanced Solana development concepts.

## Resources

### Official Documentation
- [Rust Book](https://doc.rust-lang.org/book/)
- [Solana Documentation](https://docs.solana.com/)
- [Anchor Documentation](https://www.anchor-lang.com/)

### Community
- [Solana Discord](https://discord.gg/solana)
- [Solana Stack Exchange](https://solana.stackexchange.com/)

### Tools
- [Solana Playground](https://beta.solpg.io/) - Online IDE for Solana development
- [Solana Explorer](https://explorer.solana.com/) - Blockchain explorer

## Contributing

Feel free to open issues or submit pull requests if you find any errors or want to add more content!

## License

This project is open source and available under the [MIT License](LICENSE).
