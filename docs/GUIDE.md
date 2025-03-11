# Cardano NFT Minting Platform Guide

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [Environment Setup](#environment-setup)
- [Setting Up Redis](#setting-up-redis)
- [Anvil API Integration](#anvil-api-integration)
- [Policy Configurations](#policy-configurations)
- [API Routes](#api-routes)
- [UI Components](#ui-components)
- [Pages](#pages)
- [Running Your Application](#running-your-application)
- [Security Considerations](#security-considerations)
- [Additional Resources](#additional-resources)

## Introduction

This guide demonstrates how to build a Next.js application for minting NFTs on the Cardano blockchain using the Anvil API. Whether you're creating a marketplace, a collection minting site, or just exploring blockchain integration with modern web frameworks, this project provides a solid foundation.

The application includes:

* Wallet connection with Eternl via the Weld library
* NFT minting with proper metadata formatting
* Transaction handling and status monitoring
* Clean UI with appropriate error states and loading indicators

## Prerequisites

Before beginning this project, ensure you have the following prerequisites in place:

* Node.js 18+ and npm/yarn
* Redis installed and running locally (for transaction storage)
* An Anvil API key 
* Some familiarity with Next.js and blockchain concepts

### Required Knowledge

* **Next.js**: You should understand the basics of Next.js applications, particularly the App Router structure and the Server/Client Components model.
* **TypeScript**: This project uses TypeScript for type safety.
* **Blockchain Concepts**: Basic understanding of how blockchains work, particularly Cardano's UTXO model and how NFTs function within it.

### Setting Up Your Environment

Ensure you have Node.js installed:

```bash
node --version
# Should show v18.0.0 or higher
```

Verify Redis is installed and can be started:

```bash
redis-server --version
# Should show Redis server version information
```

## Project Setup

Let's start by creating a new Next.js project with TypeScript:

```bash
npx create-next-app@latest cardano-minting-app
cd cardano-minting-app
```

When prompted during setup, select these options:
* TypeScript: Yes
* ESLint: Yes
* Tailwind CSS: Yes
* App Router: Yes
* Import alias: Yes (default @/*)

### Install Required Dependencies

```bash
npm install @ada-anvil/weld redis
```

For development, add ESLint and Prettier:

```bash
npm install -D eslint-config-prettier prettier
```

### Configure ESLint and Prettier

Create a `.prettierrc` file:

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

Update `.eslintrc.json`:

```json
{
  "extends": ["next/core-web-vitals", "prettier"],
  "rules": {
    "no-unused-vars": "warn",
    "no-console": ["warn", { "allow": ["warn", "error"] }]
  }
}
```
