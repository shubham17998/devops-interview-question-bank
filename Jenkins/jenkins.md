# Jenkins Interview Questions

## Pipeline Types

Q: What are the types of Jenkins pipelines?
Ans: Declarative Pipeline (structured, `pipeline { }` block syntax, easier to read/validate) and Scripted Pipeline (full Groovy DSL, more flexible/imperative, harder to maintain at scale).

Q: Difference between Freestyle, Pipeline and Multibranch jobs?
Ans: **Freestyle** — a simple, UI-configured job with build steps, no code/version control, limited to one linear sequence. **Pipeline** — a single Jenkinsfile-defined pipeline (stored as code, supports stages/parallelism/conditionals). **Multibranch Pipeline** — automatically discovers branches/PRs in a repo and creates/runs a pipeline job for each one that contains a Jenkinsfile.

Q: Difference between Declarative and Scripted Pipeline?
Ans: Declarative uses a fixed, structured syntax (`pipeline { agent { } stages { stage('x') { steps { } } } }`) that's easier to read, validate, and lint, with built-in support for things like `post` blocks and `options`. Scripted is pure Groovy (`node { stage('x') { ... } }`), giving full programmatic control (loops, custom functions, complex logic) at the cost of readability and stricter validation.

Q: What is a Multibranch Pipeline?
Ans: A Jenkins job type that scans a repository for branches (and optionally PRs) containing a Jenkinsfile, and automatically creates/removes a corresponding pipeline job for each — so branching in the repo tracks 1:1 with pipelines in Jenkins without manual job creation.

Q: What are Parallel Stages?
Ans: Stages within a `parallel { }` block that execute concurrently instead of sequentially — e.g., running unit tests, linting, and a security scan at the same time to reduce total pipeline duration.

Q: What are the limitations or disadvantages of Jenkins?
Ans: Requires significant self-hosting/maintenance overhead (patching, plugin management, scaling agents); plugin ecosystem quality/security varies widely and plugin conflicts are common; Groovy-based Scripted Pipelines can become messy/hard to maintain; no built-in multi-tenancy — a controller outage or resource exhaustion affects all jobs; UI/UX feels dated compared to newer CI platforms; and scaling to very large orgs typically needs careful agent/queue management.

Q: What is the difference between using a single Jenkins CI/CD pipeline versus multiple pipelines?
Ans: A single pipeline centralizes everything (simpler mental model, one place to look), but couples unrelated changes together, creates a shared bottleneck/blast radius, and can get unwieldy as scope grows. Multiple pipelines (per service/team/repo) isolate failures and allow independent, faster iteration, at the cost of more infrastructure to maintain and potential duplication of common logic (mitigated with shared libraries).

Q: What issues can arise from using a single pipeline versus multiple pipelines?
Ans: With a single pipeline: one team's failing/slow stage blocks or slows everyone else's changes, a bad change can have a wider blast radius, and the pipeline becomes a monolith that's hard to reason about/change safely. With too many separate pipelines: duplicated boilerplate across pipelines (unless shared libraries are used), harder to enforce consistent standards, and more moving parts to maintain overall.

## Agents & Nodes

Q: What is a Jenkins Agent?
Ans: A machine (physical, VM, or container) that connects to the Jenkins Controller and actually executes build/pipeline steps, keeping the Controller free from running workloads directly.

Q: What is a Jenkins Node?
Ans: The general term for any machine in a Jenkins setup capable of running builds — includes the Controller itself (if configured to run builds, generally discouraged) and all connected Agents.

Q: Difference between Node and Agent?
Ans: "Node" is the broader Jenkins term for any machine registered to run builds (technically includes the controller). "Agent" specifically refers to a worker machine that executes jobs on behalf of the Controller — in modern usage the terms are largely used interchangeably to mean a worker/executor.

Q: How do you configure Jenkins agents?
Ans: Statically via Manage Jenkins → Nodes → New Node (specify labels, executors, remote root directory, launch method — SSH, JNLP/inbound agent, etc.), or dynamically via a cloud plugin (Kubernetes plugin, EC2 plugin, Docker plugin) that provisions ephemeral agents on-demand per build and tears them down afterward.

Q: Can a pipeline use multiple agents?
Ans: Yes — each `stage` can declare its own `agent { }` block, so different stages run on different agent labels/types (e.g., build on a Linux agent, a separate stage on a Windows agent for platform-specific testing).

Q: How do you acquire multiple nodes for one specific build?
Ans: Use the `parallel` step combined with multiple `node('label') { }` blocks (Scripted) or multiple stages each with their own `agent` (Declarative) within the same pipeline run, so several agents work on different parts of the same build concurrently.

Q: What happens if the Jenkins Controller goes down while builds are running on Agents?
Ans: Running builds on Agents typically continue their current step, but lose contact with the Controller for logging/state updates and cannot proceed past a step requiring Controller coordination; once the Controller comes back, it reconnects to Agents but builds that were mid-flight during the outage are often marked as failed/aborted since the Controller lost track of their state — this is why the Controller itself needs high availability (or at least fast, monitored recovery) in production setups.

## Pipeline Fundamentals

Q: What is the role of a Jenkinsfile?
Ans: A text file (checked into source control, typically named `Jenkinsfile`) defining the entire pipeline as code — stages, steps, agents, environment, post-actions — enabling version-controlled, reviewable, reproducible CI/CD definitions instead of UI-configured jobs.

Q: How do you configure a Jenkins pipeline for smooth deployments?
Ans: Use declarative stages with clear separation (build → test → scan → deploy), gate production deploys behind manual approval (`input` step) or automated checks, use parameters/environment-specific config rather than hardcoded values, implement proper error handling (`post { failure { } }`) with notifications, and keep deployment steps idempotent so re-runs are safe.

Q: Give an example Jenkins pipeline.
Ans:
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'mvn clean package' }
        }
        stage('Test') {
            steps { sh 'mvn test' }
        }
        stage('Deploy') {
            steps { sh './deploy.sh' }
        }
    }
    post {
        failure {
            mail to: 'team@example.com', subject: 'Build Failed', body: "${env.BUILD_URL}"
        }
    }
}
```

Q: Explain your pipeline workflow? Explain your end-to-end CI/CD pipeline?
Ans: A typical flow: developer pushes code → SCM webhook triggers the Jenkins pipeline → checkout source → build (compile/package) → run unit tests and static code analysis (SAST) → build and scan a container image (Trivy) → push the image to a registry (ECR/Artifactory) → deploy to a lower environment (dev/staging) automatically → run integration/smoke tests → gate production deploy behind manual approval → deploy to production (rolling/blue-green) → run post-deploy health checks → notify the team of success/failure at each critical stage.

Q: How would you implement an option to start a build from a specific stage instead of the beginning?
Ans: Use pipeline parameters (e.g., a `choice` parameter for "start stage") combined with `when` conditions on each stage checking that parameter, so stages before the chosen start point are skipped (`when { expression { params.START_STAGE == 'deploy' || ... } }`) — or split the logical stages into separate, independently-triggerable pipelines/jobs if the granularity needed is coarse.

Q: Write a Jenkins pipeline script for a Terraform deployment.
Ans:
```groovy
pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }
    stages {
        stage('Init') {
            steps { sh 'terraform init' }
        }
        stage('Plan') {
            steps { sh 'terraform plan -out=tfplan' }
        }
        stage('Approval') {
            steps { input message: 'Apply this Terraform plan?' }
        }
        stage('Apply') {
            steps { sh 'terraform apply -auto-approve tfplan' }
        }
    }
}
```

## CI/CD

Q: Why do we use Jenkins?
Ans: It's a mature, highly extensible (2000+ plugins) open-source automation server for building, testing, and deploying software continuously — supports virtually any language/toolchain, integrates with almost every SCM/artifact/cloud/notification system, and can be self-hosted for full control over CI/CD infrastructure.

Q: Where do you store artifacts?
Ans: Dedicated artifact repositories — Artifactory, Nexus, AWS S3, or a container registry (ECR/Docker Hub) for images — rather than on the Jenkins Controller/Agent filesystem, which isn't durable or meant for long-term storage.

Q: What are artifacts?
Ans: Build outputs that need to persist beyond the pipeline run — compiled binaries, JAR/WAR files, container images, test reports, packaged releases — anything downstream stages, deployments, or audits need to reference later.

Q: Should you store build artifacts in Artifactory or in S3?
Ans: Artifactory (or Nexus) is purpose-built for artifact management — versioning, dependency resolution (acts as a Maven/npm/Docker proxy too), retention policies, and rich metadata/search — generally preferred for language-package artifacts. S3 is a fine, cheaper option for simpler needs (raw build outputs, logs, less structured artifacts) or when you don't want to run/pay for a dedicated artifact management service.

Q: What are upstream and downstream jobs?
Ans: An **upstream** job is one that triggers another job upon completion; the triggered job is its **downstream** job — used to chain pipelines (e.g., a "build" job triggers a "deploy" job downstream).

Q: How do you migrate Jenkins pipelines to GitHub Actions?
Ans: Map Jenkinsfile stages to GitHub Actions jobs/steps, replace Jenkins agents/labels with `runs-on` runner specs, convert Jenkins credentials to GitHub Secrets, replace Jenkins plugins with equivalent Marketplace Actions, translate `parallel`/`when`/`post` blocks to GitHub Actions' `strategy.matrix`/`if`/`needs` and job-level `if: failure()` conditions, and validate behavior incrementally (run both pipelines in parallel for a transition period before fully cutting over).

Q: How do you implement rollback in Jenkins?
Ans: Keep the previous release artifact/image tag available (immutable versioned artifacts), and have a rollback stage/parameterized job that re-deploys that specific prior version — for Kubernetes, this can be as simple as `kubectl rollout undo` or re-applying the prior manifest/Helm release version, triggered by a "rollback" pipeline or parameter.

Q: How do you implement approval before deployment?
Ans: Use the `input` step in the pipeline (`input message: 'Deploy to production?'`), optionally restricted to specific approver groups, which pauses pipeline execution until a human approves (or aborts) before the production deployment stage proceeds.

Q: Does a Jenkins pipeline trigger if code is pushed to a branch other than the one it's configured to build?
Ans: For a standard single-branch Pipeline job configured against a specific branch, no — pushes to other branches won't trigger it. A **Multibranch Pipeline** is the way to have Jenkins automatically build any/all branches (or ones matching a pattern) containing a Jenkinsfile.

## Administration

Q: How do you upgrade Jenkins?
Ans: Back up the `JENKINS_HOME` directory (config, jobs, plugins) first, check plugin compatibility with the target version, then upgrade via the package manager (`apt`/`yum`), the WAR file, or the Docker image tag — ideally testing the upgrade on a staging Jenkins instance restored from a prod backup before applying to production.

Q: How do you upgrade plugins in Jenkins?
Ans: Via Manage Jenkins → Plugins → Updates (upgrade individually or all at once), after backing up `JENKINS_HOME`; upgrade plugins with known dependencies together, and restart Jenkins afterward. In larger setups, test plugin upgrades on a staging instance first since plugin updates are a common source of breakage.

Q: How is user management handled in Jenkins?
Ans: Via the built-in user database, or integrated with an external identity provider (LDAP, Active Directory, SAML/OIDC SSO plugins) — combined with the Matrix-based or Role-based Authorization Strategy plugin to control per-user/group permissions on folders/jobs.

## Security & Secrets

Q: How do you secure credentials in Jenkins?
Ans: Store all secrets in the built-in **Credentials** store (encrypted at rest), scoped to the minimum folder/job that needs them, referenced in pipelines via `credentials()`/`withCredentials` rather than hardcoded — never printed to console output (Jenkins auto-masks known credential values in logs).

Q: How do you rotate secrets without downtime?
Ans: Update the secret's value in the credentials store (or external secrets manager it's synced from) under the *same* credential ID — the pipeline references the ID, not the value, so the next run automatically picks up the new secret with no pipeline/code changes needed; for actively-used long-lived connections, coordinate the rotation with a graceful reload/restart of the consuming service.

Q: How do you prevent secret leakage in logs and artifacts?
Ans: Rely on Jenkins's automatic credential masking in console output, avoid `echo`-ing secrets or passing them as visible command-line arguments (use env vars/files instead), scrub/exclude secrets from build artifacts before archiving, and run periodic secret-scanning (gitleaks/trufflehog) over both the repo and build logs.

Q: How do you secure Terraform execution from Jenkins?
Ans: Grant the Jenkins agent/pipeline only a scoped IAM role (via instance profile or short-lived assumed-role credentials, not static keys) with least-privilege permissions for exactly the resources it manages; store the Terraform state remotely with encryption and locking (S3 + DynamoDB); require plan review/approval before apply in production; and never expose `terraform plan`/`apply` output containing secrets in plain console logs.

## Optimization & Monitoring

Q: How do you optimize pipeline execution time?
Ans: Parallelize independent stages, cache dependencies between builds (Docker layer caching, package manager caches), use incremental builds where the toolchain supports it, right-size/pre-warm agents to avoid cold-start overhead, split large monolithic pipelines into smaller ones triggered only when relevant paths change, and avoid unnecessary `sleep`/polling in favor of event-driven triggers.

Q: How do you monitor pipeline failures?
Ans: Configure `post { failure { } }` blocks to send notifications (Slack/email/PagerDuty), track build history/trends via Jenkins's built-in dashboards or export metrics (via the Prometheus plugin) to Grafana, and set up alerting on failure-rate thresholds rather than relying on someone noticing a red build manually.

Q: How do you optimize a slow pipeline?
Ans: Profile which stage is actually slow (Jenkins Blue Ocean/stage timing view), then apply targeted fixes: parallelize independent stages, cache dependencies, use faster/larger agents for CPU-bound steps, skip unnecessary steps via `when` conditions, and trim unnecessary artifact archiving/log verbosity.

Q: How do you fix a broken CI/CD pipeline?
Ans: Check the console log for the exact failing step/error first; determine if it's a code issue (test failure), environment issue (agent/dependency unavailable), or config issue (recent Jenkinsfile/credential change); roll back the triggering change if it's a regression; and if it's an infra issue, restore/repair the failing agent/service and re-run.

Q: Why would you use the re-run option?
Ans: To retry a build after a transient failure (flaky test, temporary network blip, agent hiccup) without needing a new commit/trigger — useful for confirming whether a failure is a real regression or just noise, without cluttering history with an empty "retry" commit.

## GitHub Actions / GitLab CI

Q: What is workflow_dispatch?
Ans: A GitHub Actions trigger that adds a manual "Run workflow" button (and API endpoint) to trigger a workflow on demand, optionally with typed input parameters.

Q: Where does GitLab CI/CD run?
Ans: Jobs run on **GitLab Runners** — agents (shared, group, or project-specific) that pick up jobs defined in a repo's `.gitlab-ci.yml` and execute them, either GitLab-hosted (SaaS shared runners) or self-hosted or your own infrastructure.

Q: What do you know about GitLab Runners?
Ans: They're the execution agents for GitLab CI/CD, registered to a GitLab instance/project and configured with an "executor" (shell, Docker, Kubernetes, etc.) determining how jobs actually run; can be shared across the whole instance, scoped to a group, or dedicated to a single project, and support tags to control which runner picks up which jobs.

## Java Application Deployment

Q: What is Apache Tomcat, and why would you use it?
Ans: A servlet container/lightweight application server that runs Java web applications (Servlets, JSPs) — used to host and serve Java web apps packaged as WAR files, when you don't need a full Java EE application server.

Q: What is the difference between Maven and Gradle?
Ans: Both are Java build tools/dependency managers. **Maven** uses declarative XML (`pom.xml`) with a fixed, convention-based lifecycle — simpler, more standardized, but less flexible. **Gradle** uses a Groovy/Kotlin DSL (`build.gradle`) that's programmable and more flexible, with incremental builds and better caching, generally offering faster build times for larger projects.

Q: What is the difference between deploying a WAR file and a JAR file?
Ans: A **WAR** (Web Application Archive) packages a web application meant to be deployed inside an external servlet container/application server (Tomcat, JBoss). A **JAR** (Java Archive), especially an executable "fat/uber JAR" (common with Spring Boot), bundles the application with an embedded server so it can run standalone via `java -jar app.jar` without a separate application server.
