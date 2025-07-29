# Voting Smart Contract Documentation

## Overview
This is a voting smart contract built on the **Internet Computer (IC)** blockchain platform, implemented in **Rust** using the **Candid** interface. The contract enables users to create, edit, vote on, and end proposals in a decentralized manner, making it suitable for applications like decentralized governance, community decision-making, or scenarios such as DAO voting or rule-setting for a mahjong parlor reservation system. Data is stored in IC's **stable storage** (`StableBTreeMap`) to ensure persistence across contract upgrades.

## Core Functionality
The contract provides the following key features:
1. **Create Proposal**: Users can create a proposal with a description and active status.
2. **Query Proposals**: Retrieve details of a specific proposal or the total number of proposals.
3. **Edit Proposal**: The proposal owner can modify the description or active status.
4. **Vote on Proposal**: Users can vote on active proposals (Approve, Reject, or Pass), with restrictions to prevent duplicate voting.
5. **End Proposal**: The proposal owner can deactivate a proposal.
6. **Check Proposal Status**: Determine the outcome of a proposal based on voting results (APPROVED, REJECTED, PASSED, UNDECIDED, or NO_PROPOSAL).

## Data Structures
- **Proposal**: Stores proposal details, including:
  - `description`: A string describing the proposal.
  - `approve`, `reject`, `pass`: Vote counts (u32) for each option.
  - `is_active`: Boolean indicating if the proposal is open for voting.
  - `voted`: List of Principals (user IDs) who have voted.
  - `owner`: The Principal of the proposal creator.
- **CreateProposal**: Input structure for creating or editing a proposal, containing `description` and `is_active`.
- **VoteTypes**: Enum for voting options (Approve, Reject, Pass).
- **VoteError**: Enum for error handling (e.g., AlreadyVoted, ProposalNotActive, Unauthorized, NoProposal, UpdateError, VoteFailed).

## Key Functions
1. **get_proposal(key: u64) -> Option<Proposal>**
   - Queries a proposal by its unique key.
   - Returns `None` if the proposal does not exist.
2. **get_proposal_count() -> u64**
   - Returns the total number of proposals stored.
3. **create_proposal(key: u64, proposal: CreateProposal) -> Option<Proposal>**
   - Creates a new proposal with the provided key and details.
   - Sets the caller as the owner and initializes vote counts to zero.
4. **edit_proposal(key: u64, proposal: CreateProposal) -> Result<(), VoteError>**
   - Allows the proposal owner to update the description or active status.
   - Returns an error if the caller is not the owner or the proposal does not exist.
5. **end_proposal(key: u64) -> Result<(), VoteError>**
   - Deactivates a proposal, preventing further votes.
   - Only the owner can call this function.
6. **vote(key: u64, choice: VoteTypes) -> Result<(), VoteError>**
   - Allows a user to vote on an active proposal.
   - Prevents duplicate votes and voting on inactive proposals.
   - Updates the corresponding vote count (approve, reject, or pass).
7. **get_proposal_status(key: u64) -> String**
   - Returns the proposal’s status based on votes:
     - Requires at least 5 votes to determine an outcome.
     - A majority (50% or more) of votes determines if the proposal is APPROVED, REJECTED, or PASSED.
     - Returns UNDECIDED if no majority is reached or fewer than 5 votes.
     - Returns NO_PROPOSAL if the key is invalid.

## Storage
- **Stable Storage**: Uses `StableBTreeMap` from the `ic_stable_structures` crate to store proposals persistently.
- **Memory Management**: Utilizes `MemoryManager` with `VirtualMemory` to manage storage, ensuring data persists across canister upgrades.
- **Storable Implementation**: The `Proposal` struct implements `Storable` and `BoundedStorable` for serialization and deserialization, with a maximum size of 100 bytes (configurable via `MAX_VALUE_SIZE`).

## Use Cases
- **Decentralized Governance**: Suitable for DAOs or community-driven decision-making, such as voting on rules or proposals.
- **Mahjong Parlor Reservation System**: Can be adapted to vote on operational rules (e.g., reservation policies, pricing) for a mahjong parlor, aligning with your interest in autonomous reservation systems.
- **Community Polls**: Useful for lightweight polling in communities or organizations.

## Deployment
- **Environment**: Deploy on the Internet Computer using the `dfx` CLI tool.
- **Dependencies**:
  - `candid`: For Candid interface and serialization.
  - `ic-cdk`: For IC-specific functionality (e.g., `ic_cdk::caller`).
  - `ic-stable-structures`: For stable storage.
- **Steps**:
  1. Install `dfx` and set up an IC project.
  2. Compile and deploy the Rust canister using `dfx deploy`.
  3. Interact with the contract using Candid UI or custom frontends.
- **Configuration**: Adjust `MAX_VALUE_SIZE` in the code if larger proposal descriptions are needed.

## Usage Example
1. **Create a Proposal**:
   ```bash
   dfx canister call voting_canister create_proposal '(1, record { description = "Set mahjong parlor reservation fee to $10"; is_active = true })'
