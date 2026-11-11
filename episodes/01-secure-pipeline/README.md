# Episode 01: A green pipeline is not a secure pipeline

A green pipeline proves only that the gates you defined passed. If there are no security gates, green means "it compiled and deployed".

**Animated page:** https://hassankx.github.io/simplified-devops/01-secure-pipeline/

## The seven stages

| Stage | What it is for | Checks | Example tools |
|-------|----------------|--------|---------------|
| Plan | Decide what could go wrong | Threat model, security requirements | Design review, tickets |
| Code | Catch it before it merges | Secrets scan, SAST, IaC scan | Gitleaks, SonarQube, Checkov |
| Build | Scan what you ship | Dependency scan, image scan, SBOM | Trivy, Syft |
| Test | Attack the running app | DAST, API security tests | OWASP ZAP |
| Release | Prove where it came from | Sign image, attest SBOM | Cosign |
| Deploy | The cluster checks for itself | Verify signature, pod policy | Kyverno or OPA Gatekeeper |
| Operate | Keep watching after release | Runtime detection, monitoring, rescans | Falco, Prometheus, Trivy |

## What fails the build

Adding scanners is the easy part. The decision that matters is what each finding does. This is an example policy; tune it to your own risk.

| Finding | Action | Why |
|---------|--------|-----|
| Any secret | Block | There is no safe severity for a leaked credential. Rotate it. |
| Critical CVE, fix available | Block | The remedy is an upgrade. |
| CVE with no fix | Exception | Blocking achieves nothing. Record an owner and an expiry date. |
| New vulnerability in changed code | Block | The author fixes it while it is fresh. |
| Old findings in untouched code | Ticket | Failing every build on old debt teaches people to ignore the gate. |
| High-severity IaC misconfiguration | Block | Cheapest to fix while it is still a text file. |
| Unsigned image | Reject | The cluster refuses it at admission. |
| Runtime alert | Page on-call | It already happened. Fast detection is what is left. |

## Six things that can go wrong, and which gate catches each

| What happens | Caught at | By |
|--------------|-----------|----|
| An AWS key is committed | Code | Secrets scan |
| A library with a critical CVE is added | Build | Dependency scan |
| Terraform makes a bucket public | Code | IaC scan |
| An image is pushed by hand, skipping the pipeline | Deploy | Signature verification |
| An attacker opens a shell in a running container | Operate | Runtime detection (alert, not block) |
| Nothing | Nowhere | Every gate passes and it deploys |

## Files

- [`secure-pipeline.yml`](secure-pipeline.yml): a GitHub Actions workflow with one job per stage. Copy it to `.github/workflows/` in your own repository.
- [`require-signed-images.yaml`](require-signed-images.yaml): a Kyverno policy that rejects any pod whose image was not signed by that workflow.

Neither file has been run against a real repository or cluster. Before using them: add your secrets (`SONAR_TOKEN`), variables (`SONAR_HOST_URL`, `STAGING_URL`), registry and paths, replace `your-org/your-repo`, and pin each action to a version you have checked.

## Interview questions

**The pipeline is green. What does that prove?**
Only that the gates you defined passed. A good answer names them: no secrets, no fixable critical CVEs, quality gate passed, image signed, admitted by policy.

**SAST, SCA and DAST: what is the difference?**
SAST reads your own source for flaws. SCA checks third-party dependencies against known CVEs. DAST probes the running application over HTTP. They find different things, so one does not replace another.

**A developer committed an AWS key and the scan caught it. What now?**
Treat the key as compromised. Revoke and rotate it first, then check the provider's logs for use. Removing the commit comes second. Then add a pre-commit hook.

**Trivy reports a critical CVE with no fix available. Do you block the release?**
Not automatically. Check whether the vulnerable code is reachable and whether another base image avoids it. If you accept the risk, record an exception with an owner and an expiry date.

**Why sign container images, and where is the signature checked?**
A tag can be moved; a signature ties one digest to the pipeline that built it. An admission controller in the cluster checks it when a pod is created, so a deploy that skipped the pipeline is refused.

**What is an SBOM for?**
It lists every package inside an image. When a new vulnerability is announced, you search the SBOMs to learn which deployed images are affected, without rescanning everything.

**How do you add security scans without slowing developers down?**
Run scans in parallel, cache the vulnerability databases, gate only on new code and on findings that are severe and fixable, and report in the pull request.

**What does shift right mean, and why do you need it if you shift left?**
Shift left moves checks earlier, where fixes are cheap. Shift right is detection in production. You need both because some problems only exist at runtime, and new vulnerabilities appear after release.

**How do you keep policy consistent across environments?**
Write policy as code, keep it in Git, and apply the same policies to every cluster: audit mode in dev, enforce in production. Exceptions are reviewed changes.

**Which single check would you never skip?**
Secrets scanning. It is fast, and a leaked credential gives an attacker access without needing any vulnerability.
