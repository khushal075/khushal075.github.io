---
title: "Efficient agents need a workflow, not just a stronger model"
description: "A practical architecture combining rules, small ML models, language-model routing, and bounded collaboration to improve business outcomes."
date: 2026-10-30
tags: ["Applied AI", "Enterprise architecture", "Agent workflows"]
draft: false
---

A support request arrives: identify the customer, understand the issue, retrieve the right policy, recommend an action, and possibly change a subscription.

A powerful model could attempt the whole task. But should it?

Some steps are database lookups. Others are classification. A few require judgment. One may need explicit human approval. Treating every step as a reasoning problem makes the system harder to price, test, and operate.

The architecture goal is to complete the business task correctly, within its time, cost, and authority limits. Model intelligence is one resource in that design.

This article proposes a workflow; it does not describe features already implemented in CrewOS or claim measured performance gains.

## 1. Define success before choosing models

For the support example, success might mean a correct recommendation, evidence from the applicable policy, no unauthorized account changes, and an acceptable response time.

Measure the outcome rather than merely counting completed model calls. Useful metrics include accepted resolutions, escalation rate, correction rate, latency, and cost per correctly resolved request.

Distinguish a fixed workflow from an agent that chooses its next actions. Use a predefined path when the task is predictable; add dynamic planning when the work genuinely needs it. Anthropic describes this distinction and recommends increasing complexity only when necessary. [Building effective agents](https://www.anthropic.com/research/building-effective-agents)

## 2. Establish a trusted execution contract

Before any model runs, validate the input, authenticate the caller, and establish authorized tenant scope.

Create a run record with a stable ID, objective, allowed tools, output requirements, deadline, and spending limits. Persist progress where recovery requires it.

The application owns these boundaries. A model may propose actions, but it cannot grant itself permissions or redefine the tenant it serves. Treat retrieved documents and tool responses as evidence, not instructions that can override authorization.

![A bounded hybrid AI workflow combining rules, ML routing, language models, collaboration, verification, and approval](/images/efficient-agent-workflow.svg)

## 3. Use rules and small ML where they fit

A language model is unnecessary for every decision.

| Step | Candidate mechanism | Why consider it? |
| --- | --- | --- |
| Validate account ownership | Authorization rules and database lookup | The answer comes from trusted records |
| Route a known support category | TF-IDF with logistic regression or a linear classifier | A measurable baseline for labeled text |
| Estimate escalation risk from structured features | Logistic regression or gradient-boosted trees | Handles a defined prediction task |
| Retrieve related policies | Search or embeddings with enforced access filters | Narrows the evidence supplied to the model |
| Explain a policy in context | A language model | Requires language interpretation and synthesis |
| Change a subscription | Authorized application code | Requires controlled side effects |

Small ML models are useful when the labels, inputs, and evaluation criteria are sufficiently stable. They still require training data, maintenance, and monitoring.

A classifier score is not automatically a reliable probability. Check calibration on held-out data and examine performance for relevant customer and task groups. Scikit-learn documents calibration methods and their limitations. [Probability calibration](https://scikit-learn.org/stable/modules/calibration.html)

If uncertainty is high, abstain and route to a more capable path or a person. Do not invent a universal confidence threshold.

## 4. Route language work by difficulty and risk

Start with a cheaper model for a bounded extraction or summarization task when evaluation shows it meets the requirement. Reserve stronger reasoning models for tasks that need them.

Routing should consider task type, available evidence, observed failure patterns, latency, and permitted data destinations. Switching providers can change the data-handling boundary, not just the price.

Escalate when the result fails a check, conflicts with evidence, or exceeds the simpler path's demonstrated capability. A model saying it is confident is not sufficient evidence.

Evaluate the routing system end to end. Money saved on initial calls can disappear through retries, reviews, and corrections.

## 5. Make collaboration purposeful and bounded

Multiple models can contribute different roles: one extracts facts, another investigates an independent question, and a stronger model synthesizes evidence. A separate reviewer may identify unsupported claims.

Multiple roles do not require different model families, and different models do not guarantee independent errors. Shared training patterns or the same misleading source can lead them to agree incorrectly.

Parallelize work that is genuinely independent. For a research task, one worker might inspect the technical evidence while another examines operating costs. Give each worker a narrow objective, scoped tools, evidence requirements, and a deadline.

Return structured findings rather than entire conversation histories. The coordinator should track missing evidence, conflicts, and unfinished work before producing a conclusion.

Anthropic's multi-agent research report describes the benefits and additional coordination challenges of parallel research. Its results are specific to that system, not a general guarantee for every multi-agent design. [Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)

Avoid collaboration when the work is tightly sequential or cheap enough to solve reliably in one pass. Coordination has a cost.

## 6. Verify before committing an action

Validate output structure, required fields, identifiers, authorization, and business invariants with application code wherever possible.

For a policy recommendation, check whether cited evidence exists and applies to the customer's situation. An LLM reviewer can add a useful signal; agreement between models is not proof.

Separate proposing a change from executing it. Require approval where the business process demands it, and bind that approval to the exact action and parameters. Revalidate authorization at execution time.

Protect external writes with operation IDs and downstream deduplication where supported. A network timeout does not establish whether a change happened, so retries must account for ambiguous outcomes.

## 7. Budget the whole workflow

Limit total spending, elapsed time, tool calls, retries, and parallel workers. Reserve capacity for verification and delivery rather than allowing planning to consume the entire budget.

When a limit is reached, return an explicit incomplete state, save recoverable progress, or escalate. Do not turn partial evidence into an apparently complete answer.

Record the route selected, model versions, tool outcomes, latency, and cost. Avoid indiscriminately logging sensitive prompts or customer data.

## 8. Improve against a baseline

Compare a simple single-model workflow with routed and collaborative alternatives on representative tasks. Include ambiguous requests, missing evidence, provider failures, and attempted cross-tenant access.

Track quality alongside cost and latency. Review false confident routes and unnecessary escalations. Monitor changes in traffic and labels; retrain or recalibrate small models when evidence justifies it.

The architecture decision is whether the additional machinery improves the business outcome enough to justify its complexity.

An efficient agent system assigns each step to a mechanism it can justify: rules for known constraints, small ML for defined predictions, language models for interpretation, collaboration for separable work, and people for decisions that require human authority.
