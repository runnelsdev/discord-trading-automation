# Discord Trading Automation — Architecture

> Scope: workflow architecture, subscription lifecycle, and routing concepts only. No proprietary trading strategy logic is documented in this repository.

## System Architecture

```mermaid
flowchart LR
    Sub[Subscription Provider] --> Webhook[Webhook Receiver]
    Webhook --> State[Subscription State Store]
    State --> Router{Tier Router}
    Router --> Tier1[Tier 1 Access]
    Router --> Tier2[Tier 2 Access]
    Router --> Tier3[Tier 3 Access]
    Tier1 --> Discord[Discord Role Sync]
    Tier2 --> Discord
    Tier3 --> Discord
    Discord --> Dash[Operational Dashboard]
    State --> Comms[Lifecycle Comms]
    Comms --> Member[Member]
```

## Subscription Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Trial
    Trial --> Active: Payment success
    Trial --> Lapsed: Trial ends
    Active --> PastDue: Payment failed
    PastDue --> Active: Recovery
    PastDue --> Canceled: Grace period ends
    Active --> Canceled: Member cancels
    Canceled --> [*]
    Lapsed --> Active: Reactivation
```

## Tier Routing Logic (Conceptual)

```mermaid
flowchart TD
    Event[Subscription Event] --> Parse[Parse Plan / Add-ons]
    Parse --> Determine{Determine Tier}
    Determine -->|Plan A| T1[Tier 1 Roles]
    Determine -->|Plan B| T2[Tier 2 Roles]
    Determine -->|Plan C + Add-ons| T3[Tier 3 Roles]
    T1 --> Sync[Sync Discord Roles]
    T2 --> Sync
    T3 --> Sync
    Sync --> Audit[Audit Log]
    Audit --> Dashboard[Community Health Dashboard]
```

## Session & Communications

```mermaid
sequenceDiagram
    participant Stripe
    participant System as Automation
    participant Discord
    participant Member

    Stripe->>System: subscription.created
    System->>Discord: Assign tier role
    System->>Member: Welcome DM
    Stripe->>System: subscription.updated
    System->>Discord: Sync role changes
    System->>Member: Plan-change notice
    Stripe->>System: subscription.deleted
    System->>Discord: Revoke tier role
    System->>Member: Off-boarding comms
```
