---
title: "An AI workflow needs execution boundaries"
description: "How CrewOS separates submission, model execution, tenant authorization, and result access—and what remains to build."
date: 2026-10-29
tags: ["Applied AI", "Architecture decisions"]
draft: false
---

Calling a model is one step in a system. Accepting work, deciding who owns it, routing it, and authorizing the result are separate responsibilities.

[CrewOS](https://github.com/khushal075/crewos) is a Python foundation for exploring those responsibilities. Its current implementation executes a fixed two-step LLM workflow: analysis followed by processing. Output from the first step is included in the next model request.

## Separate submission from execution

The asynchronous API validates a request, authenticates the tenant, records task ownership in Redis, and publishes work to a tenant-specific Celery queue. A matching worker performs the model calls and stores the outcome for authorized polling.

```text
Request → tenant authentication → task ownership → tenant queue
                                                   ↓
Authorized result ← Redis result ← worker ← two-step LLM workflow
```

This keeps slow model calls outside the submission request. It does not guarantee fast completion: queue contention, model availability, and inference time still determine when a result arrives.

## Tenant context is not authorization

A tenant header selects a scope. In the current implementation, a matching configured bearer token authorizes access at tenant level. Result reads also check task ownership.

Queue routing makes workload placement explicit, but it is not a complete security boundary. Shared Redis and model infrastructure can still create contention and need their own deployment decisions. Individual-user permissions and SSO are separate capabilities, not consequences of having tenant queues.

## Give retries one owner

CrewOS gives Celery responsibility for selected transient inference failures, with bounded backoff and jitter. Model-client automatic retries are disabled so the two layers do not silently multiply attempts.

A repeated model request may generate again and incur additional cost. A retry policy is therefore part of the execution contract, not simply an availability setting.

## Keep model integration replaceable

The model adapter uses an OpenAI-compatible endpoint, with Ollama as a local inference option. Endpoint configuration separates the service from a particular model deployment.

Local inference still requires model availability and suitable compute. Compatibility of an endpoint does not establish the reasoning quality of the model behind it.

## State the current boundary honestly

This foundation demonstrates workflow delivery, tenant-aware execution, context forwarding, and controlled result access. It is not yet a general autonomous agent runtime.

Tool-using loops, persistent memory, durable workflow checkpoints, approval flows, and multi-agent evaluation remain planned. Verification against a deterministic model fixture demonstrates execution behavior; it does not measure real-model quality.

That distinction matters in an architecture portfolio. The useful evidence is the system that can be inspected today, alongside a clear account of the decisions still ahead.
