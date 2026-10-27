# Mimamori (見守り): AI CCTV Video Search on AWS EKS

[![ci](https://github.com/codewithmuh/ai-cctv-video-search-aws-eks/actions/workflows/ci.yml/badge.svg)](https://github.com/codewithmuh/ai-cctv-video-search-aws-eks/actions/workflows/ci.yml)
[![release](https://github.com/codewithmuh/ai-cctv-video-search-aws-eks/actions/workflows/release.yml/badge.svg)](https://github.com/codewithmuh/ai-cctv-video-search-aws-eks/actions/workflows/release.yml)
[![License: learning only, no commercial use](https://img.shields.io/badge/License-learning%20only%2C%20no%20commercial%20use-orange.svg)](LICENSE)

**Search your cameras in plain words.** Mimamori watches many camera streams and lets you search them the way
you would ask a person: "a red car at the gate", "someone carrying a ladder". It sends alerts the same way. It
runs on Amazon EKS with its own Kubernetes operator, models shared across cameras on one GPU, and SLOs.

> **Objects and actions, never identities.** No face recognition: faces are blurred in everything it stores,
> and the demo uses public footage or cameras you own.

**Status: phase 0, building in public.** The repository, CI and the API's plumbing are done; the demo cameras and choosing the models is next.
See the [roadmap](docs/roadmap.md).

## Part 5 of AI on EKS

Five production AI apps on Amazon EKS, one industry each. Every part adds one new layer of Kubernetes on top of
the parts before it.

| Part | Project | Industry | New on Kubernetes |
|---|---|---|---|
| 1 | [Heya](https://github.com/codewithmuh/ai-virtual-staging-aws-eks): AI virtual staging | Real estate | EKS basics: ALB Ingress with HTTPS, HPA, Pod Identity, then Argo CD |
| 2 | [Karute](https://github.com/codewithmuh/ai-medical-scribe-aws-eks): AI medical scribe | Healthcare | Security: NetworkPolicies, VPC endpoints, External Secrets, WebSockets |
| 3 | [Kisekae](https://github.com/codewithmuh/ai-virtual-try-on-aws-eks): AI virtual try-on | Fashion | GPUs: Karpenter, KEDA on SQS, Spot, scale to zero |
| 4 | [Fukikae](https://github.com/codewithmuh/ai-video-dubbing-aws-eks): AI dubbing in your own voice | Media | GPU pipelines: Argo Workflows, GPU sharing, RBAC |
| **5** | **Mimamori: AI CCTV video search (this repo)** | **Security & retail** | **Operators (CRD), batched model serving, GPU metrics, SLOs** |

## The demo

1. Eight camera feeds play on a wall.
2. Search in plain words: "a red car at the gate", "someone carrying a ladder". The matching moments appear as
   clips, each with one line describing it.
3. Add a standing alert: "someone leaves a bag behind". When it happens, Slack gets the clip and the description.
4. `kubectl apply` a new camera and watch it join the wall.

Then the platform demo: Grafana shows frames per second per camera, GPU load and alert latency against the SLO,
while the camera count goes from 4 to 32.

## What it proves

| Question an interviewer asks | How Mimamori answers it |
|---|---|
| "How do you extend Kubernetes?" | A `Camera` custom resource and a small operator: apply a camera and its pipeline appears |
| "How do you serve models efficiently?" | A model server that batches frames from many cameras on one GPU |
| "How do you scale on GPU load?" | GPU metrics from the DCGM exporter in Prometheus drive the autoscaling |
| "How do you know it works?" | SLOs for alert latency and dropped frames, with burn-rate alerts, runbooks and Grafana dashboards as code |
| "What does a camera cost?" | Cost per camera per hour from 4 to 32 cameras, published |

The full design, rules and the numbers it will publish are in [docs/scope.md](docs/scope.md).

## The stack

| Layer | Choice |
|---|---|
| AI | Open-vocabulary object detection and image–text embeddings (models chosen in ADRs); Claude (vision) to check and describe matches |
| App | Next.js web, FastAPI API, one ingest worker per camera, a model server, an alerter, PostgreSQL with pgvector, MediaMTX to replay demo footage as RTSP cameras |
| Kubernetes | `Camera` CRD and a Python operator (kopf), batched model serving, KEDA, DCGM exporter, Prometheus and Grafana, SLOs, Argo CD |
| AWS | GPU instances (g5 or g6), RDS PostgreSQL, S3 for clips (7-day lifecycle), Pod Identity, all in Terraform |

## Before you start

- GPU quota, as in [part 3](https://github.com/codewithmuh/ai-virtual-try-on-aws-eks#before-you-start). Ask early:
  approval can take days.
- An Anthropic API key, and a Slack workspace for the alerts.
- Demo footage: scripts download public footage whose license allows it, or use cameras you own.
- Every phase publishes its measured cost per hour and ends with a teardown that leaves nothing running.

## Quick start

Requirements: [uv](https://docs.astral.sh/uv/), [pre-commit](https://pre-commit.com/) and Docker, or just open
the **Dev Container**.

```bash
make setup    # install dependencies and git hooks
make check    # lint, type-check and test (same as CI)
make up       # the API on :8000

curl localhost:8000/healthz          # {"status":"ok"}
curl localhost:8000/readyz           # {"status":"ready"}

make down
```

Run `make` with no arguments to list every command.

## Repository layout

```
services/api/      FastAPI service: src/api (config, logs, probes, metrics), tests/, Dockerfile. The
                   camera, search and alert endpoints go here as they are built.
docs/              scope, roadmap, architecture, threat model, ADRs, runbooks
compose.yaml       local run with production-like hardening
Makefile           every command you need
```

The web app, Terraform and Kubernetes manifests arrive with their phases in the [roadmap](docs/roadmap.md).

## How changes flow

1. Open a pull request with a [Conventional Commit](https://www.conventionalcommits.org/) title.
2. CI must pass: lint, types, tests, secret and vulnerability scans, image build and smoke test.
3. release-please keeps a Release PR open. Merging it tags the version and publishes a signed image to GHCR with an
   SBOM.

## Production-grade checklist

Every item must pass before v1.0.

- [ ] Code quality: linters, formatters, type checks and pre-commit pass
- [ ] Tests: unit + integration, ≥ 80% coverage, a smoke test and a load test with published numbers
- [ ] CI: every PR runs lint, tests, build and scans; `main` is protected
- [ ] Security: no secrets in git, least privilege, scanned and signed images, SBOM, threat model
- [ ] Infrastructure as code: remote state, `make up` / `make down`, cost estimate, clean teardown
- [ ] Releases: semantic versions, changelog, images in a registry
- [ ] Observability: logs, metrics, traces, dashboards as code, alerts linked to runbooks
- [ ] Reliability: probes, graceful shutdown, timeouts/retries/idempotency, tested backups, written SLOs
- [ ] Docs: README with quick start, architecture, ADRs, runbooks, contributing, license
- [ ] Developer experience: one-command setup, Dev Container or Compose, seed data
- [ ] Demo: screenshots, a short demo video, a write-up

## License

**Learning only. Commercial use is not allowed.** You may clone, run, modify and deploy this project to learn, use
it to teach at a school or in free workshops, and show it in your portfolio with credit. You may not sell it, run it
as a service, use it in company or client work, or use it in paid or monetised courses and videos without written
permission. The full terms are in [LICENSE](LICENSE).

For a commercial license, or if you are unsure whether your use is allowed: **contact@codewithmuh.com**

© 2026 Muhammad Rashid ([CodeWithMuh](https://youtube.com/@codewithmuh)), Senior Platform Engineer ·
[codewithmuh.com](https://codewithmuh.com) · [GitHub](https://github.com/codewithmuh) ·
[LinkedIn](https://www.linkedin.com/in/muhammad-rashid-daha/)
