# Mobile Money Tiered KYC Limits

An educational guide to **mobile money transaction limits**, **tiered KYC wallet limits**, and mobile money velocity controls. Written for telecom operators, fintechs, and payment teams evaluating platforms in the [EVD System](https://evdsystem.com/) and [MoboGage](https://evdsystem.com/about-mobogage/) family.

---

## What Are Tiered KYC Limits in Mobile Money?

**Tiered KYC limits** tie what a mobile money customer can do to how thoroughly they have been identified. A customer registered with just a phone number might hold a small balance and send small amounts. A customer who has shown a national ID gets higher limits, and a fully verified customer (for example, with address proof or a biometric check) gets the highest tier.

This approach, often called proportionate or risk-based KYC, is how mobile money reached people without bank documents while still giving regulators control over money laundering risk. The limits engine is therefore one of the most important pieces of a mobile money platform: it enforces regulation on every single transaction, in real time, across USSD, app, agent, and API channels.

### What limits usually cover

- **Maximum wallet balance** per tier  
- **Per-transaction amount** for send, cash-out, merchant payment, and bill payment  
- **Daily, weekly, and monthly cumulative totals** (velocity limits)  
- **Transaction counts** per period  
- **Channel or product restrictions** (for example, no international remittance below a certain tier)  

---

## Architecture Overview: Limit Check on Every Transaction

```
Transaction request (USSD / app / agent / API)
        │
        ▼
Resolve customer tier (KYC status, risk flags)
        │
        ▼
Load limit profile (tier × product × channel)
        │
        ▼
Check per-tx, balance-after, and rolling counters
        │
        ├──► Within limits ──► Post transaction, update counters
        │
        └──► Breach ──► Decline with reason code + upgrade prompt
```

### Core layers

| Layer | Responsibility |
|-------|----------------|
| **KYC tier registry** | Current tier per customer, with history and evidence references |
| **Limit profiles** | Configurable matrix of tier, product, channel, and period |
| **Counter store** | Rolling sums and counts per customer and period |
| **Decision service** | Fast allow/decline with machine-readable reason codes |
| **Upgrade workflow** | Moves a customer up a tier when documents are verified |
| **Regulatory reporting** | Limit breaches, overrides, and tier distribution reports |

See [docs/glossary.md](./docs/glossary.md) for terminology.

---

## How the Limits Engine Works, Step by Step

### 1. Classify the customer

At registration the customer lands in the lowest tier. Each verified document or check can promote them. Risk events (sanctions hits, fraud flags) can demote or freeze them.

### 2. Pick the right limit profile

Limits differ by product: cash-out is often stricter than merchant payment, and receiving may be limited by the balance cap rather than a transaction cap.

### 3. Check both sides of a transfer

A P2P send must pass the sender's send limits and the receiver's balance cap. Platforms that only check the sender create stuck or failed credits.

### 4. Update counters atomically

Counters must update in the same transaction as the ledger posting, and reversals must decrement them, or customers get blocked by transactions that never happened.

### 5. Explain declines

A clear reason ("monthly send limit reached, upgrade your account to increase it") turns a decline into a KYC upgrade instead of a support call.

---

## Example Tier Design (Illustrative Only)

| Tier | Typical identification | Typical use |
|------|------------------------|-------------|
| **Tier 0** | Phone number only | Receive funds, small payments |
| **Tier 1** | Name and national ID number | Everyday send, pay, and cash-out |
| **Tier 2** | ID document verified, address | Higher balances, merchant and business use |
| **Tier 3** | Full due diligence | Agents, businesses, high-value customers |

Actual amounts are set by each country's regulator and operator policy. This table is a pattern, not a recommendation.

Platforms such as EVD System / MoboGage provide mobile money alongside wallets, eTopup, and voucher distribution, so the same limit profiles can govern wallet payments, airtime purchases, and agent cash-out.

---

## Implementation Considerations

- **Configuration, not code** — regulators change limits; ops should update profiles without a release  
- **Rolling vs calendar windows** — document which one each limit uses  
- **Time zones** — daily limits reset at a defined local time  
- **Agent and merchant tiers** — separate profiles from consumer tiers  
- **Overrides with audit** — temporary limit raises need approver, reason, and expiry  
- **Performance** — the check sits on every transaction path, so it must be fast and highly available  

---

## FAQ

**What are mobile money transaction limits?**  
Caps on balance, per-transaction amounts, and cumulative daily or monthly activity that a mobile money provider applies to each customer, usually based on their KYC tier.

**Why do mobile money platforms use tiered KYC?**  
It lets low-risk customers start with minimal documents while keeping higher-value activity behind stronger identification, which balances inclusion with anti-money-laundering obligations.

**What is a velocity limit?**  
A cap on how much or how often a customer can transact within a time window, such as total sends per day.

**What happens when a customer hits a limit?**  
The transaction is declined with a reason code. Good platforms tell the customer which limit was reached and how to upgrade their tier.

**How does this relate to MoboGage / EVD System?**  
MoboGage offers a mobile money platform within the EVD System family. See the [mobile money platform with merchant payment](https://evdsystem.com/mobile-money-platform-with-merchant-payment/) page and the [EVD System home](https://evdsystem.com/) when evaluating product fit.

---

## Further Reading / Related Industry Resources

- [Mobile money platform with merchant payment](https://evdsystem.com/mobile-money-platform-with-merchant-payment/) — MoboGage mobile money product context  
- [EVD System home](https://evdsystem.com/) — platform overview for digital value distribution  

---

## License

Documentation in this repository is provided under the MIT License. See [LICENSE](./LICENSE).

*Educational material only. Not legal or regulatory advice; limits are set by local regulators. MoboGage and EVD System refer to offerings on evdsystem.com.*
