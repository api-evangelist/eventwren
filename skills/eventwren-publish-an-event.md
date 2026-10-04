---
generated: '2026-09-26'
method: generated
name: publish-an-event
description: Create an Eventwren account, hand the owner the account link to fund it, post an event and poll until moderation decides.
api: openapi/eventwren-openapi.yml
operations: [createAccount, getAccount, createPost, getPost]
source: >-
  Grounded in the flow on https://eventwren.com/developers/ (steps 1-4); operationIds
  verified in openapi/eventwren-openapi.yml.
---

# Publish an event

1. `createAccount` (POST /v1/accounts) with name, email and `accept_terms: true` — only when the human owner accepts the terms. Store the `api_key`; it is shown once.
2. Give the returned `account_url` to your human: an agent cannot verify email, add a card or top up. The link expires after an hour. `getAccount` shows the balance.
3. `createPost` (POST /v1/posts) with title, body, timezone, starts_at, ends_at, topics and location. Expect 202 `queued`; the fee ($1.00) is charged now and refunded on a low-quality rejection. A 402 with `for_human: true` means the owner must top up — hand them `error.account_url`, then retry.
4. `getPost` (GET /v1/posts/{id}) until status is `published`, `review` or `rejected`. Honour `Retry-After` / `moderation.estimated_decision_at`; a `starting` model can take about 20 minutes.

Rules: no idempotency key exists, so do not blindly retry a POST that may have been queued — check `getPost` first. Read `getPolicy` before posting; abuse costs 10x the price and three strikes ban the account. See conventions/eventwren-conventions.yml.
