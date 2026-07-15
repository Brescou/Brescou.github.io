---
title: "Shipping LangGraph agents past the demo"
description: "Why I built langgraph-agent-stack — a self-hosted template for multi-agent systems with cost caps, pack canaries, and Kubernetes ops baked in."
date: 2026-07-15
draft: false
---

Most LangGraph demos die the same way: they work in a notebook, then someone asks about budgets, rollouts, metrics, or “can we run this on our cluster?” and the answer is a shrug.

I built [langgraph-agent-stack](https://github.com/Brescou/langgraph-agent-stack) for that gap — a deployable starting point for teams who want to **own** a multi-agent stack on Kubernetes instead of stopping at a PoC.

## The problem

Getting agents to answer a question is the easy part. Production asks harder ones:

- What happens when a run burns through the LLM budget?
- How do you ship a new pack version without flipping 100% of traffic?
- Who owns auth, rate limits, structured logs, and Prometheus?
- Can you rebuild and sign the image on every merge?

Generic “FastAPI around LangGraph” templates usually leave those as TODOs. Managed platforms solve some of them — at the cost of control and lock-in.

## What this template is (and isn’t)

**It is:** a MIT-licensed, self-hosted template. Hardened FastAPI API, domain packs, Helm (KEDA autoscaling on active pipelines), Terraform stubs for GKE / EKS / AKS, rate limiting, input validation, cost tracking, CI security scans, GHCR images with SBOM + Cosign on `main`.

**It isn’t:** a hosted control plane. Multi-tenant identity, OAuth2/OIDC, and billing stay as *your* layer — the built-in `API_KEY` is a shared Bearer for internal / single-tenant use. That trade-off is intentional: keep the core honest about what it owns.

If you want a managed control plane and don’t want to run infra, LangGraph Platform is a legitimate choice. This repo is for the opposite case: your cluster, your observability, your cost data, no vendor lock-in.

## Production concerns that ship with the box

A few things I refused to leave as “exercise for the reader”:

**Per-run cost caps.** Pack budgets return HTTP `402` when a run overruns. That turns “LLM spend” into an API contract instead of a surprise invoice.

**Pack versioning and canaries.** Domain packs can carry multiple versions with traffic weights and sticky sessions. You can canary a pack the same way you’d canary a service — not by hoping the prompt change is fine.

**Ops path that isn’t CPU-only.** The Helm chart defaults to KEDA on `active_pipelines`. Agent workloads spike on concurrency, not average CPU; autoscaling should match that.

**Supply chain on `main`.** Images go to GHCR with SBOM + Cosign. Boring, necessary, usually missing from agent repos.

There’s also a mock LLM provider for zero-cost exploration and CI — useful when you want to exercise the API and pack routes before wiring Anthropic, OpenAI, Bedrock, or the rest.

## Shape of the stack

At a high level:

```
Client → FastAPI (auth · rate limit · validation)
              → PackRegistry / control_plane policies
              → domain_packs/* (LangGraph workflows)
              → agents/* (reusable nodes)
              → core/* (LLM · memory · cost · observability)
```

Packs are the unit of product behavior (research, meeting prep, support triage, …). Regulated-adjacent packs (HR, legal, finance) stay **off by default** until you opt in after a compliance checklist — calling them returns `403` until then.

## Who it’s for

ML / data / MLOps engineers who need a real agent service on their own Kubernetes, and prefer build-vs-buy toward **build** for the control plane.

Who should skip it: teams happy on a managed platform, or anyone looking for a notebook tutorial. The README is the source of truth for quick start, pack catalogue, and deploy paths.

## Try it

```bash
git clone https://github.com/Brescou/langgraph-agent-stack.git
cd langgraph-agent-stack
uv sync --extra anthropic
cp .env.example .env
# optional: LLM_PROVIDER=mock for zero-cost exploration
uv run uvicorn api.main:app --reload
```

Repo: [github.com/Brescou/langgraph-agent-stack](https://github.com/Brescou/langgraph-agent-stack)

If you’re industrializing agentic systems and care about the unglamorous 90% — budgets, canaries, Helm, CI — this is the template I wish I’d had earlier. Issues and PRs welcome.
