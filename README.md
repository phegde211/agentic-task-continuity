# Task Continuity in Agentic AI Flows

> [!IMPORTANT]
> **Problem statement — public preview.** The full protocol specification
> has been authored and is being worked upon for IETF submission for
> Internet-Draft. See [Status and access](#6-status-and-access).

## 1. Context

In an agentic AI flow, multiple autonomous agents collaborate to complete a
single task. The user entitles a front-end agent with the task; that agent
delegates parts of it to specialist agents, which may delegate further. The
resulting delegation tree routinely spans administrative boundaries —
participants operated by different organisations, under different policies,
with no shared coordinator.

Agent-to-agent transport is well served by existing protocols. What no
protocol provides is a task lifecycle that survives delegation: every hop
creates a fresh, unrelated task, and the task as *one accountable piece of
work* ceases to exist at the first boundary.

## 2. Illustration

A user asks a travel agent to book a vacation. The travel agent engages a
flight agent, a hotel agent, and a cab agent — each operated by a different
company. The flight agent in turn engages airline agents; the hotel agent,
an aggregator.

```
 User: "book a vacation"
    |
    v              task-17                    task-62
 [Travel Agent] ----------> [Flight Agent] ----------> [Airline-X Agent]
    |                            |                task-91
    |                            +--------------------> [Airline-Y Agent]
    |              task-23                    task-44
    +---------------------> [Hotel  Agent] ----------> [Aggregator Agent]
    |              task-31
    +---------------------> [Cab    Agent]

        Every delegation mints a new, unrelated task identifier.
        Nothing ties task-91 back to "the vacation".
```

The user then cancels the vacation:

```
 CANCEL task-17 ---> [Flight Agent]   X . . . . . . . . . . . . . . .
                     signal ends here    The airline agents never hear
                                         it: seats are ticketed, cards
                                         are charged.

 Airline-Y had already issued a non-refundable ticket. Which effects
 were committed? Which remain refundable, and until when? There is no
 path for those answers to reach the accountable party.
```

## 3. The three consequences

They compound:

1. **Correlation is impossible.** Each agent's records are accurate, but
   they share no common key. Even with the full cooperation of every
   participant, "the vacation" cannot be reconstructed as a single history.
2. **Cancellation stops at the first hop.** The travel agent can cancel with
   its immediate peer. It has no protocol means of reaching — or even
   naming — the agents below. Work continues beneath the point where the
   signal died.
3. **Committed effects are invisible.** Cancellation is a request about the
   future. It says nothing about what was already done, what remains
   reversible, and for how long.

## 4. By design, not by accident

The [A2A Protocol v1.0.0](https://a2a-protocol.org/latest/specification/)
(Linux Foundation) states its operating principle in §1.2, *Guiding
Principles*:

> **Opaque Execution:** Agents collaborate based on declared capabilities
> and exchanged information, without needing to share their internal
> thoughts, plans, or tool implementations.

Opacity is sound — an agent's choice of downstream providers is frequently
its business. But it has consequences: task identifiers are generated
independently at each hop, and `CancelTask` (§3.1.5) binds only the directly
addressed peer.

Adjacent mechanisms do not close the gap:

- **Credential revocation** prevents *new* actions. It neither stops work in
  progress nor reports what was already committed.
- **Distributed tracing** is telemetry: sampled, regenerated at
  organisational boundaries, and carrying no obligation on any recipient to
  act.

## 5. Requirements on any solution

| # | Requirement | Property |
|---|-------------|----------|
| R1 | **Continuity** | The task remains one identity across every hop, conveying no authority and no task content. |
| R2 | **Reach** | A stop signal takes effect at depths the originator cannot see or name — with no central registry. |
| R3 | **Accounting** | The accountable party learns what stopped, what was committed but remains reversible (and until when), and what is permanent. |
| R4 | **Truthful uncertainty** | An unreachable participant surfaces as *unknown* — never as "stopped". |
| R5 | **Opacity preserved** | No agent discloses its downstream relationships in ordinary operation, yet the chain remains reconstructible to an authenticated, authorised auditor. |

The requirements are in deliberate tension: R2 appears to demand exactly the
registry R5 forbids, and R4 demands visibility that R5 restricts. Resolving
all five simultaneously is the design problem.

## 6. Status and access

A complete protocol specification addressing R1–R5 has been authored. I am
actively working toward submitting it to the IETF as an individual
Internet-Draft and am engaging with the relevant people to take it through
that process. This repository will be updated once the draft is submitted.

The full specification is held in a private repository:
[phegde211/secret_vault](https://github.com/phegde211/secret_vault)
(the link returns 404 unless you have been granted access).

> [!NOTE]
> **To request access**, reach out to the author at
> **prabhat.hegde@gmail.com** with your GitHub email ID, and permission
> can be granted based on the request.

---
*Author: Prabhat Hegde*
