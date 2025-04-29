# BitStake: Stacks Layer 2 Staking and Governance Protocol

**BitStake** is a decentralized, modular protocol designed for the **Stacks blockchain**, enabling native **STX staking** with **tiered rewards**, **flexible lock periods**, and **on-chain governance**. It provides users with incentivized staking mechanics and democratic decision-making via proposals and votes.

## Features

- **STX Staking with Lock Periods**: Users can stake STX tokens with optional locking durations to receive reward multipliers.
- **Tiered Membership System**: Dynamic tier assignment based on staking volume, unlocking higher reward rates and feature access.
- **On-Chain Governance**: Stakeholders can create and vote on proposals to shape protocol evolution.
- **Cooldown-based Unstaking**: Ensures security and fairness by introducing an unstaking cooldown window.
- **Emergency Controls**: Includes contract pausing and emergency flags for secure and controlled management.

## Protocol Design

### Token

- **Token Name**: `ANALYTICS-TOKEN`  
- **Purpose**: Placeholder for future reward distribution or analytics incentives.

### Constants

| Constant              | Description                          |
|-----------------------|--------------------------------------|
| `CONTRACT-OWNER`      | Address of deployer / admin          |
| `base-reward-rate`    | Default annualized reward rate (5%)  |
| `bonus-rate`          | Bonus based on lock duration (1%)    |
| `cooldown-period`     | Unstaking cooldown period (24 hrs)   |
| `minimum-stake`       | Minimum stake amount (1,000,000 uSTX)|

## Staking Logic

### `stake-stx (amount uint) (lock-period uint)`

Stake STX tokens with optional lock period. Locking enhances reward multiplier:

- No lock: `1.0x`
- 1 month: `1.25x`
- 2 months: `1.5x`

Additionally, tier-levels grant:

- Tier 1 (≥1M uSTX): `1.0x`
- Tier 2 (≥5M uSTX): `1.5x`
- Tier 3 (≥10M uSTX): `2.0x`

> Effective multiplier = `tier-multiplier * lock-multiplier`

---

## Unstaking

### `initiate-unstake (amount uint)`

- Starts a cooldown timer for the specified amount.
- Prevents immediate withdrawal to ensure security.

### `complete-unstake`

- Can only be called after the cooldown period.
- Transfers staked amount back to user and clears stake record.

## Governance

### `create-proposal (description, voting-period)`

- Allows users with ≥ 1M `voting-power` to submit proposals.
- Proposals must have valid description length (10–256 chars).
- Voting window is configurable (100–2880 blocks).

### `vote-on-proposal (proposal-id, vote-for)`

- Stakeholders can vote *for* or *against* an active proposal.
- Weighted by `voting_power` held by the user.

## Data Structures

### `UserPositions`

Stores per-user stats:

- `stx-staked`
- `analytics-tokens`
- `voting-power`
- `tier-level`
- `rewards-multiplier`

### `StakingPositions`

Tracks staking session data:

- `amount`
- `start-block`
- `lock-period`
- `cooldown-start`
- `accumulated-rewards`

### `TierLevels`

Tier access defined by:

- `minimum-stake`
- `reward-multiplier`
- `features-enabled` list of booleans

### `Proposals`

Stores governance metadata:

- `creator`, `description`
- `start-block`, `end-block`
- `votes-for`, `votes-against`
- `minimum-votes`
- `executed` flag

## Admin Functions

- `pause-contract` / `resume-contract`: Temporarily disables/enables operations.
- `initialize-contract`: Must be called by `CONTRACT-OWNER` to set up tier levels.

## Read-Only Views

| Function                | Description                         |
|-------------------------|-------------------------------------|
| `get-contract-owner`    | Returns the contract admin          |
| `get-stx-pool`          | Returns total STX staked            |
| `get-proposal-count`    | Returns current number of proposals |

## Error Codes

| Code        | Meaning                              |
|-------------|---------------------------------------|
| `u1000`     | Not authorized                        |
| `u1001`     | Invalid protocol action               |
| `u1002`     | Invalid amount                        |
| `u1003`     | Insufficient STX for requested op     |
| `u1004`     | Cooldown already active               |
| `u1005`     | No stake found for user               |
| `u1006`     | Amount below required minimum         |
| `u1007`     | Contract is currently paused          |

## Deployment & Initialization

After deploying the contract:

```clojure
(initialize-contract)
```

This sets the reward tiers and activates tier logic.

---

## Reward Calculation

Rewards are dynamically calculated based on:

- Stake amount
- Base rate
- Effective multiplier (`tier × lock`)
- Number of blocks staked

```clojure
(* (* (* stake-amount base-rate) rewards-multiplier) duration) / u14400000
```

---

## Development Notes

- Written in **Clarity**, Stacks’ smart contract language.
- Carefully leverages `unwrap!`, `asserts!`, and `map-get?` for safety.
- Modular helper functions for validation and reward logic.
- Read-only views and strict control flow to ensure transparency.

## Future Improvements

- Integration of actual utility for `ANALYTICS-TOKEN`.
- Modular upgradeability for new tier logic or lock types.
- On-chain execution of passed proposals.

## Contributing

Contributions, audits, and feedback are welcome. Please open issues or submit PRs for improvements.
