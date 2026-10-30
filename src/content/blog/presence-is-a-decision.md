---
title: "When a connection disappears, when is a user offline?"
description: "A walkthrough of the presence playground: heartbeat timeouts, stale state, and the boundary between a useful simulation and a production design."
date: 2026-10-29
tags: ["Distributed systems", "Architecture decisions"]
draft: false
---

A connection disappearing is an event. Declaring a user offline is a decision about what the system believes and what it tells other people.

The [presence playground](/playground/) makes one part of that decision visible. It starts with two application nodes and a simulated set of connected sessions. Disconnect Node A and its sessions remain counted briefly. After a three-second timeout, those sessions become offline.

> This is a browser simulation with illustrative numbers. It does not run a Redis server, establish real WebSocket sessions, or measure production performance.

## Start with the meaning, not the mechanism

“Online” might mean that a user has an open connection, recently sent a heartbeat, or is actively interacting with an application. Those meanings lead to different product behavior.

A useful architecture discussion starts by deciding which meaning the product needs. For a collaboration tool, a briefly stale indicator may be acceptable. A workflow that assigns urgent work may need a different signal and a more explicit treatment of uncertainty.

The decision record should explain the meaning of presence, the acceptable delay before an offline transition, and what happens when evidence is missing.

## What the playground actually does

The model has three states:

```text
Healthy: both nodes have connected sessions
    ↓ Disconnect Node A
Timeout pending: Node A is offline, but its sessions remain counted
    ↓ Three seconds elapse
Degraded: Node A's sessions expire; Node B remains connected
```

Moving the client slider changes the illustrative session population. Reconnecting Node A restores its sessions. Reset returns the model to 120 clients and a healthy state.

The chart shows recent simulated connected-session counts. It is useful for seeing the transition; it is not a latency benchmark or a capacity-planning result.

## The timeout is a tradeoff

A shorter timeout detects missing heartbeats sooner. It can also classify temporary network interruptions as offline transitions more quickly. A longer timeout tolerates more interruption but leaves stale presence visible for longer.

Three seconds is a demonstration parameter here, not a recommendation for a production service. A production choice needs evidence about heartbeat frequency, network behavior, scheduling delays, and the experience users expect.

It also needs a distinction between a session and a person. A user may have several devices or browser tabs. Losing one session does not necessarily mean that the user is offline.

## A diagram is not a delivery guarantee

The visual topology includes Redis Pub/Sub and TTL to explain coordination and expiry. Animated lines show relationships, not proof that an update was durably delivered.

A production design still needs explicit answers:

- How does a reconnecting client obtain the current state?
- What happens to updates missed during a disconnection?
- How are simultaneous sessions for the same user combined?
- Which component owns expiry and publishes the resulting transition?
- How are tenant boundaries enforced during subscriptions and fan-out?

These are contracts to define and verify, rather than details to infer from a box labeled “Redis.”

## Make the decision inspectable

The purpose of this playground is to connect an architecture choice to observable behavior. A visitor can trigger a failure and see the timeout policy take effect.

That is also a useful standard for a technical decision record: state the constraint, describe the chosen behavior, identify what remains uncertain, and show how the decision can be tested.

[Try the simulation](/playground/), or [explore the presence-service repository](https://github.com/khushal075/presence_service) for the separate implementation.
