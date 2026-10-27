# Threat model

Lightweight [STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats) review.
Revisit it when you add an external integration, a data store, or a new trust boundary.

## Assets

- Camera footage of real places and the people in them
- Camera credentials (RTSP)
- Alerts, which people act on
- GPU capacity and service availability
- Build and release pipeline (images, signing identity)

## Threats and mitigations

| STRIDE | Threat | Mitigation | Status |
|---|---|---|---|
| Spoofing | Forged release images | Keyless cosign signatures tied to this repo's workflow | ✅ |
| Tampering | Compromised GitHub Action or dependency | Actions pinned to SHAs, Dependabot, Trivy scans, SBOM | ✅ |
| Repudiation | Unknown who changed what | Protected `main`, required reviews, signed release provenance | ✅ |
| Information disclosure | Secrets committed to git | gitleaks in pre-commit and CI; `.env` ignored | ✅ |
| Elevation of privilege | Container breakout | Non-root, read-only, no capabilities, minimal base image | ✅ |
| Denial of service | Request floods exhaust the service | Rate limits at the edge (AWS WAF on the ALB) and per user in the API | ⏳ planned |
| Denial of service | Abuse runs up the AI or GPU bill ("denial of wallet") | Per-user daily limits, a global daily budget, alerts at 80% | ⏳ planned |
| Tampering | Prompt injection through uploaded content | Model output is schema-validated and never triggers an action on its own | ⏳ planned |
| Information disclosure | The system is used to identify or track people | Rule R1: no face-recognition model; faces blurred before any frame or clip is stored | ⏳ planned |
| Information disclosure | Camera credentials leak | RTSP credentials in Secrets Manager via External Secrets; only ingest pods can read them | ⏳ planned |
| Spoofing | Something other than an ingest pod connects to the cameras | NetworkPolicies: only ingest pods may reach the camera network | ⏳ planned |
| Tampering | A false alert causes a bad decision | Every alert carries the clip, the description and a confidence; a person confirms it | ⏳ planned |
| Elevation of privilege | The operator's permissions are abused | The operator's RBAC is limited to Cameras, and the Deployments, ConfigMaps and Secrets in one namespace | ⏳ planned |

## Open risks

List accepted risks with an owner and a review date.
