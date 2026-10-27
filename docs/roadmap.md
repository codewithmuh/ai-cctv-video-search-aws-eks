# Roadmap

Built in public, phase by phase. Every phase ends with measured results, not claims. See [scope.md](scope.md).

## Phase 0: the system, locally (one GPU)

- [x] Repository from the production template: CI, scans, signed releases, FastAPI plumbing (probes, metrics,
      JSON logs, graceful shutdown)
- [ ] ADRs: detection and embedding models (accuracy, speed, license), model server, pgvector, demo footage and its
      licenses
- [ ] Demo cameras: public footage replayed as RTSP streams with MediaMTX
- [ ] Ingest worker: sample frames, skip frames with no motion, blur faces before storing (rule R1)
- [ ] Search: text → embedding → nearest frames → Claude checks and describes the top matches
- [ ] Alerter: standing alerts in plain words, sent to Slack with the clip
- [ ] Web: camera wall, search, alerts

## Phase 1: on EKS

- [ ] Terraform: VPC, EKS, Karpenter, RDS PostgreSQL with pgvector, S3 with a 7-day lifecycle, Pod Identity
- [ ] `Camera` custom resource and a Python operator (kopf): apply a camera, get its ingest worker, config and secret
- [ ] Model server with dynamic batching across cameras on one GPU
- [ ] Argo CD for everything; camera credentials from Secrets Manager via External Secrets

## Phase 2: observable and measured

- [ ] Prometheus, Grafana and DCGM GPU metrics; dashboards as code
- [ ] SLOs for alert latency and dropped frames, with burn-rate alerts and runbooks
- [ ] Scale test from 4 to 32 cameras: GPUs needed, latency, cost per camera per hour
- [ ] Drill: lose the GPU node; ingest buffers and catches up with no footage lost

## Phase 3: v1.0

- [ ] All 11 areas of the production-grade checklist
- [ ] Results table in the README, demo video and write-up
