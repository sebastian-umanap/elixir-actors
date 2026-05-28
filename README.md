# Elixir actor model in Athena

**Author:** Sebastian Alberto Umaña Peinado — `s.umanap` — 202013778
**Course:** ISIS-4220 Software Verification

Specification and verification of the Elixir actor model. Built in layers; each file loads the previous one.

## How to run

```bash
athena main.ath
```

`main.ath` loads everything and re-claims every theorem. If it runs clean, every `Theorem:` shows up without any errors.

## Files

| File | What it has |
|---|---|
| `message.ath` | `domain Message` (kept abstract). |
| `mailbox.ath` | `Mailbox` datatype, `enqueue` / `head-mb` / `tail-mb` / `is-empty?`. FIFO theorem. |
| `actor.ath` | `Actor` structure (`pidOf`, `stateOf`, `mboxOf`), `deliver`, `run`, `behavior`. P1..P6. |
| `system.ath` | `System` with `lookup`/`update` and McCarthy's axioms, `send-to`, `step`. S1..S6. |
| `main.ath` | Loads everything and re-claims every theorem. |

## Theorems and what property they cover

### (1) State isolation

| Theorem | What it says |
|---|---|
| S1 | `~ p=q ==> lookup (send-to sys p m) q = lookup sys q` |
| S2 | `~ p=q ==> lookup (step sys p) q = lookup sys q` |
| S5 | no `send-to` ever changes any actor's state |

### (2) Exclusive internal modification of state

| Theorem | What it says |
|---|---|
| P5 | the new state after `run` is exactly `behavior s m` |
| P6 | `deliver` never changes the state |
| S6 | `step` on a different PID never changes another actor's state |

### (3) Message processing

| Theorem | What it says |
|---|---|
| P2 | `deliver` enqueues exactly one message at the end |
| P3 | `run` preserves the PID |
| P4 | `run` consumes exactly one message |
| P5 | state advances exactly by `behavior` |
| S3 | locality of send |
| S4 | locality of step |

### (4) Mailbox order (FIFO)

| Theorem | What it says |
|---|---|
| `fifo-two` | enqueueing m1 then m2 on empty gives head = m1 |
| `enqueue-preserves-head` | enqueueing on a non-empty mailbox does not change the head |

### (5) Type of actor system

Following De Koster, Van Cutsem & De Meuter [1], actor systems fall into four families: *Classic Actors*, *Active Objects*, **Processes**, and *Communicating Event-Loops*. Elixir (inherited from Erlang [3]) is in the **Processes** family: each actor is an independent lightweight process with

- its own identity (PID) — `domain Pid` and `pidOf`, used by P3 and `run-pid-helper`;
- private state (share-nothing) — `domain State` and `stateOf`, protected by S1, S2, S5, S6, P6;
- a per-process FIFO mailbox — `Mailbox` and `fifo-two`;
- one-message-at-a-time processing — captured by the deterministic semantics of `run` and by P4, P5.

This is different from *Communicating Event-Loops* (e.g. AmbientTalk), where a **vat** holds several objects that share one event queue. The term *vat* belongs to that family, not to Elixir; in Elixir the unit of isolation is the process itself.

## What I left out and why

The project statement allows leaving out language details that are not needed to describe the actor system. I omitted:

- **Supervision trees, `link`/`monitor`, restart, OTP/GenServer.** Fault-tolerance patterns built on top of the basic actor model; the model itself (process, mailbox, send, receive) is what I verify.
- **Distribution between nodes, selective `receive` with timeouts, hot code reload.** Orthogonal features that don't change isolation or FIFO order.
- **Elixir's type system, language-level pattern matching, real scheduler.** Implementation details; the proofs don't depend on them.
- **Message contents.** `Message` is abstract — none of the properties need to look inside.
- **Concrete `behavior`.** Left uninterpreted; we only prove that `run` applies it exactly to `(state, msg)`, which is what "exclusive internal modification" really means.

## Theorems proved

15 in total (plus 1 helper lemma), all verified by loading `main.ath`.

- Mailbox: `fifo-two`, `enqueue-preserves-head` (2)
- Actor: `P1-pid`, `P1-state`, `P2`, `P3`, `P4`, `P5`, `P6` + helper `run-pid-helper` (7 + 1)
- System: `S1`, `S2`, `S3`, `S4`, `S5`, `S6` (6)

## References

Numbering follows the project statement (`project.pdf`).

[1] De Koster, J., Van Cutsem, T., & De Meuter, W. (2016). *43 years of actors: a taxonomy of actor models and their key properties*. AGERE! 2016, pp. 31–40. Used for the classification (section 5).

[2] Bereczky, P., Horpácsi, D., & Thompson, S. (2023). *A Formalisation of Core Erlang, a concurrent actor language*. arXiv:2311.10482. Background on the underlying semantics (Elixir runs on the BEAM).

[3] Perugini, S., & Wright, D. J. (2018). *Concurrent programming with the actor model in Elixir*. Journal of Computing Sciences in Colleges, 35(5), 108–118. Main source for the Elixir semantics modeled here (PID, `send`/`spawn`/`receive`, FIFO mailbox, sequential processing).
