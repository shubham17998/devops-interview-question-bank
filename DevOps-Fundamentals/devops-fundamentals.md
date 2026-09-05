# DevOps Fundamentals Interview Questions

## DevOps Concepts
1. What is DevOps?
   A set of practices and a culture that unifies software development (Dev) and IT operations (Ops), aiming to shorten the development lifecycle and deliver high-quality software continuously through automation, collaboration, and shared ownership of the whole delivery pipeline.

2. What are the core principles of DevOps?
   Collaboration between Dev and Ops (breaking down silos), automation of repetitive tasks (build/test/deploy), continuous integration and delivery, infrastructure as code, monitoring/feedback loops, and a culture of shared responsibility, blameless postmortems, and continuous improvement.

3. What DevOps methodologies do you follow while implementing a project?
   Typically a blend: Agile/Scrum for iterative delivery, CI/CD for automated build-test-deploy, Infrastructure as Code for reproducible environments, GitOps for declarative deployment management, and Shift-Left practices (testing/security earlier in the pipeline) — chosen and combined based on the team/project's actual needs rather than dogmatically following one framework.

4. Explain the complete CI/CD lifecycle from scratch.
   Developer commits code → CI server (Jenkins/GitHub Actions) triggers on push → checkout → build/compile → run unit tests and static analysis → package artifact (binary/container image) → push to an artifact/image registry → CD process deploys to a lower environment (dev/staging) automatically → run integration/smoke tests → optional manual approval gate → deploy to production (rolling/blue-green/canary) → post-deploy health checks and monitoring → automatic rollback if health checks fail.

5. What are the stages of a software development lifecycle (SDLC)?
   Requirements gathering, design, implementation (coding), testing, deployment, and maintenance — often iterated continuously rather than strictly sequential in modern Agile/DevOps practice.

6. How does DevOps improve software delivery?
   By automating manual, error-prone steps (builds, tests, deployments), enabling faster and more frequent releases with lower risk per release, catching issues earlier via continuous testing/feedback, and improving reliability through consistent, repeatable, version-controlled infrastructure and deployment processes.

7. What is Infrastructure as Code (IaC)?
   Managing and provisioning infrastructure through machine-readable declarative/imperative configuration files (Terraform, CloudFormation, Ansible) instead of manual, click-through provisioning — enabling version control, code review, repeatability, and consistency across environments.

8. What is GitOps?
   An operating model where Git is the single source of truth for both application and infrastructure declarative state; an automated controller (e.g., ArgoCD, Flux) continuously reconciles the live environment to match what's declared in Git, and changes are made exclusively through Git commits/PRs rather than direct manual changes.

9. What is Shift-Left testing?
   Moving testing (and security/quality checks) as early as possible in the development lifecycle — running unit tests, static analysis, and security scans at commit/PR time rather than only after deployment — so defects are found and fixed when they're cheapest to address.

10. What is DevSecOps?
    The practice of embedding security practices and tooling (SAST, DAST, dependency/image scanning, secrets detection) directly into the CI/CD pipeline and development workflow from the start, rather than treating security as a separate, late-stage gate — "security as everyone's responsibility, built in, not bolted on."

## Collaboration & Process
11. How do development and operations teams collaborate in DevOps?
    Through shared ownership of the full delivery pipeline (developers involved in on-call/operations, ops involved in design/architecture reviews), common tooling (shared dashboards, shared incident channels), blameless postmortems that focus on systemic fixes, and cross-functional teams rather than hard handoffs between separate silos.

12. What challenges have you faced while implementing DevOps practices?
    Common real-world challenges: cultural resistance to shared ownership/blameless practices, legacy systems not designed for automation/IaC, toolchain fragmentation across teams, balancing deployment speed against stability/compliance requirements, and getting buy-in for the upfront investment automation requires before its payoff is visible.

13. How do you measure DevOps success?
    Track DORA metrics — deployment frequency, lead time for changes, change failure rate, and mean time to recovery (MTTR) — alongside operational metrics like uptime/SLA adherence, incident count/severity trends, and team-reported metrics like developer satisfaction/cycle time.

14. What KPIs do you track for CI/CD pipelines?
    Pipeline success/failure rate, average build/pipeline duration, deployment frequency, lead time from commit to production, change failure rate, mean time to recovery, and flaky-test rate.

15. How do you handle change management in production?
    Require peer-reviewed PRs and passing automated checks before merge, use progressive rollout strategies (canary/blue-green) with monitoring gates, require manual approval for high-risk changes, maintain a documented rollback plan for every change, and log/audit all production changes for traceability.

## Release Management
16. What is Continuous Integration?
    The practice of developers frequently merging code changes into a shared main branch, with each merge automatically triggering a build and test run — catching integration issues early and often, rather than in large, infrequent, risky merges.

17. What is Continuous Delivery?
    Extending CI so that every change that passes automated tests is automatically built into a release-ready artifact and could be deployed to production at any time — but the actual production deployment still requires a manual trigger/approval.

18. What is Continuous Deployment?
    Going one step further than Continuous Delivery — every change that passes all automated tests/checks is deployed to production automatically, with no manual approval gate at all.

19. Difference between Continuous Delivery and Continuous Deployment?
    Both automate everything up through having a production-ready, tested artifact. Continuous Delivery stops short of production and requires a human to trigger the final deploy; Continuous Deployment removes that manual gate entirely, deploying automatically whenever checks pass.

20. What deployment strategies have you used?
    Rolling updates, blue-green deployments, canary releases, and (less commonly) shadow deployments — chosen based on the risk tolerance, rollback speed needed, and resource cost the team/project can accept for a given service.

## Scaling
21. What is the key difference between Horizontal Scaling and Vertical Scaling?
    **Vertical scaling (scale up)** adds more resources (CPU/RAM) to an existing single machine — simple, but has a hard ceiling and typically requires downtime to resize. **Horizontal scaling (scale out)** adds more machines/instances running in parallel behind a load balancer — effectively limitless, improves fault tolerance (no single point of failure), but requires the application to be designed to run statelessly/distributed across multiple nodes.

> Git-specific questions have moved to [Git/git.md](../Git/git.md).

## Coding Practice

General scripting/algorithm exercises that come up alongside DevOps interviews.

### Exercise 1: Binary Search
**Objective:** Implement an O(log n) search over a sorted list.
**Task:** Write a function that takes a sorted list and a target value, and returns the index of the target (or -1 if not found) using binary search.

**Solution (Python):**
```python
from typing import List, Optional

def binary_search(arr: List[int], lb: int, ub: int, target: int) -> Optional[int]:
    while lb <= ub:
        mid = lb + (ub - lb) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lb = mid + 1
        else:
            ub = mid - 1
    return -1
```

### Exercise 2: Merge Sort
**Objective:** Implement an O(n log n) divide-and-conquer sort.
**Task:** Write a function that sorts a list using merge sort.

**Solution (Python):**
```python
from typing import List

def merge_sort(arr: List[int]) -> List[int]:
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(left: List[int], right: List[int]) -> List[int]:
    merged = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            merged.append(left[i])
            i += 1
        else:
            merged.append(right[j])
            j += 1
    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged
```

### Exercise 3: Reimplement `grep -A -B` in Python
**Objective:** Practice text processing — a very common DevOps take-home task (log parsing).
**Task:** Implement the equivalent of `grep error -A 2 -B 2 some_file` in Python: find lines matching a pattern and print them along with N lines of context before and after each match.

**Solution (Python):**
```python
def grep_with_context(filename: str, pattern: str, before: int = 2, after: int = 2) -> None:
    with open(filename) as f:
        lines = f.readlines()

    match_indexes = [i for i, line in enumerate(lines) if pattern in line]

    for idx in match_indexes:
        start = max(0, idx - before)
        end = min(len(lines), idx + after + 1)
        print("".join(lines[start:end]))
        print("--")

if __name__ == "__main__":
    grep_with_context("some_file", "error")
```
