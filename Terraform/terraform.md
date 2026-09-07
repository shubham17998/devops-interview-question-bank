# Terraform Interview Questions

## Basics
Q: What is Terraform?
Ans: An open-source Infrastructure as Code tool (by HashiCorp) that lets you define cloud/on-prem infrastructure declaratively in configuration files, then plan and apply changes to make real infrastructure match that declared state — supporting many providers (AWS, Azure, GCP, Kubernetes, etc.) through a single workflow.

Q: What language does Terraform use?
Ans: HCL (HashiCorp Configuration Language) — a declarative configuration language; Terraform also accepts an equivalent JSON syntax for machine-generated configs.

Q: What are the different Terraform file formats?
Ans: `.tf` (HCL configuration files), `.tf.json` (JSON-equivalent configuration), `.tfvars`/`.tfvars.json` (variable value files), and `.tfstate`/`.tfstate.backup` (state files, JSON format).

Q: What is terraform fmt?
Ans: A command that automatically rewrites `.tf` files to Terraform's canonical formatting style (consistent indentation/alignment) — purely cosmetic, doesn't validate logic.

Q: What is Terraform Validate?
Ans: `terraform validate` checks the configuration for internal syntax errors and basic logical consistency (e.g., referencing an undefined variable) — without contacting any provider or checking real infrastructure state.

Q: Difference between Terraform Validate and Terraform Fmt?
Ans: `fmt` only reformats code style; it doesn't check correctness. `validate` checks the configuration is syntactically and internally valid, but doesn't check style/formatting.

Q: What does terraform init do?
Ans: Initializes a working directory: downloads/installs the required provider plugins, sets up the configured backend for state storage, and initializes any modules referenced — must be run before `plan`/`apply` in a new or modified configuration.

Q: What is the purpose of the `terraform graph` command?
Ans: Outputs a visual representation (in DOT format, renderable with Graphviz) of the dependency graph between resources in the configuration — useful for understanding execution order and resource relationships.

Q: What are Terraform providers, and where are they downloaded from after `terraform init`?
Ans: Providers are plugins that implement the actual API calls to a specific platform (AWS, Azure, Kubernetes, etc.), translating Terraform resource blocks into real API operations. They're downloaded from the Terraform Registry (registry.terraform.io) by default (or a private/mirrored registry, if configured) during `terraform init`, and cached locally in `.terraform/providers`.

Q: Can multiple providers be used in a single Terraform configuration?
Ans: Yes — a single configuration can declare and use multiple providers simultaneously (e.g., AWS + Kubernetes + Cloudflare in one config), and even multiple configured instances of the same provider (e.g., two AWS regions) using provider aliases.

Q: How do you auto-approve Terraform changes without an interactive prompt?
Ans: `terraform apply -auto-approve` (skips the "yes" confirmation prompt) — used in CI/CD pipelines where interactive confirmation isn't possible, generally only after a reviewed `plan` step.

## State Management
Q: What is Terraform state and why is it needed?
Ans: A JSON file (`terraform.tfstate`) that maps your configuration's resources to their real-world provider IDs and tracks their last-known attributes. Terraform needs it to know what already exists, compute diffs (plan) between desired and actual state, and know what to update/destroy — without it, Terraform has no memory of what it previously created.

Q: Where do you store Terraform State in production?
Ans: A remote backend shared by the team — most commonly S3 (with DynamoDB for locking), Terraform Cloud/Enterprise, or Azure Blob/GCS equivalent — never just a local file, which can't be safely shared, versioned, or protected from concurrent writes.

Q: What is Terraform State Locking?
Ans: A mechanism that prevents concurrent `apply`/`plan` operations from corrupting the same state file by acquiring a lock before an operation starts and releasing it after — critical for team environments to avoid two people applying changes simultaneously and clobbering each other's state.

Q: How does DynamoDB locking work?
Ans: With an S3 backend, Terraform can use a DynamoDB table (with a primary key `LockID`) to hold a lock item during an operation — before writing to the state file, Terraform writes a lock record to DynamoDB; if a lock already exists, the operation fails/waits rather than proceeding concurrently; the lock item is removed once the operation completes.

Q: What if only S3 backend is available?
Ans: Without DynamoDB, S3-only state storage has no locking — concurrent applies can race and corrupt the state file. (Note: newer Terraform versions support native S3 conditional-write-based locking without DynamoDB, but the traditional/most common pattern still pairs S3 + DynamoDB for locking.)

Q: What happens if Terraform state is accidentally deleted?
Ans: Terraform loses all knowledge of what it previously created — running `plan` afterward would show it wanting to create everything from scratch (potentially duplicating existing real resources). Recovery: restore from a state backup/versioned S3 bucket, or manually rebuild the state using `terraform import` for each existing resource.

Q: How would you handle a Terraform state corruption?
Ans: Restore from the most recent known-good backup (S3 versioning, or the automatic `.tfstate.backup` file Terraform keeps locally), or if unavailable, rebuild the state resource-by-resource with `terraform import`, then run `terraform plan` to verify the reconstructed state matches actual infrastructure with no unexpected diffs before applying anything further.

Q: Difference between Terraform State and Workspace?
Ans: **State** is the record of resources and their attributes for one specific environment/configuration. A **Workspace** is a named, isolated instance of state within the same configuration/backend — e.g., using workspaces to manage `dev`/`staging`/`prod` state files separately from the same `.tf` code without duplicating configuration.

Q: How do you migrate Terraform state to a new backend without recreating resources?
Ans: Update the `backend` block to the new backend configuration, then run `terraform init -migrate-state`, which copies the existing state into the new backend location — the real infrastructure is untouched since Terraform is only relocating where it tracks state, not re-provisioning anything.

Q: How does Terraform prevent duplicate resource creation?
Ans: By tracking every resource it manages in the state file — on `plan`/`apply`, it compares the configuration against the state (and optionally a refresh against real infrastructure) to determine exactly what needs to be created/changed/destroyed, only creating resources that aren't already recorded as existing.

## Drift
Q: What is Terraform Drift?
Ans: A divergence between the actual state of real infrastructure and what Terraform's state file records — usually caused by manual/out-of-band changes made outside of Terraform (console click-ops, another tool).

Q: Drift vs Refresh?
Ans: **Drift** is the *condition* of real infrastructure having diverged from Terraform's recorded state. **Refresh** is the *action* Terraform takes (as part of `plan`, or explicitly via `terraform apply -refresh-only`) to query real infrastructure and update the state file to reflect the actual current attributes — refreshing is how drift gets detected/reflected in state.

Q: Example of Terraform Drift?
Ans: Someone manually resizes an EC2 instance's EBS volume from 8GB to 20GB directly in the AWS console, without going through Terraform — the next `terraform plan` will show a diff (Terraform wants to "fix" it back to 8GB per its configuration) since real infrastructure now differs from the declared config/state.

Q: How do you handle Terraform Drift?
Ans: Detect it via regular `terraform plan` runs (or `terraform plan -refresh-only`) to see the diff, then decide: either update the Terraform configuration to match the new real-world value (if the manual change was intentional/correct) and apply, or run `terraform apply` to revert the drifted resource back to the declared configuration (if the manual change was unintended).

Q: What would you do if Terraform drift is detected in production?
Ans: Investigate why the drift occurred first (who/what changed it and why) before blindly applying — if the manual change was a legitimate emergency fix, update the Terraform config to reflect the new desired state and apply from there; if it was accidental/unauthorized, revert it via `terraform apply` and address the process gap that allowed manual changes to slip through (e.g., restricting console write access).

Q: What happens if someone manually changes infrastructure managed by Terraform?
Ans: Terraform's state file becomes stale relative to reality. The next `plan`/`apply` run will detect the difference (drift) and, unless the configuration is updated to match, will attempt to revert the resource back to what's declared in the `.tf` files — potentially undoing the manual change unexpectedly.

Q: If EBS volume changed from 8GB to 20GB manually, what happens during Terraform Apply?
Ans: `terraform plan`/`apply` refreshes state, sees the real volume is 20GB but the configuration still declares 8GB, and shows a plan to resize it back down to 8GB — applying that plan would shrink the volume back to 8GB (note: EBS doesn't actually support shrinking via this path in practice; the resize would likely fail or require replacement, depending on the exact attribute changed).

Q: What will appear in the Terraform plan if you comment out a resource block that was previously applied?
Ans: Terraform will show that resource marked for **destruction** — since the resource no longer exists in the configuration but is still recorded in state, `plan` interprets its absence as "this should be removed" and proposes destroying it.

## Refresh & Import
Q: What is Terraform Refresh?
Ans: The process of reconciling the state file with real infrastructure's current actual attribute values — historically a standalone `terraform refresh` command, now typically done as part of `plan`/`apply`, or explicitly via `terraform apply -refresh-only` (updates state to reality without proposing any config-driven changes).

Q: What is terraform apply -refresh-only?
Ans: A mode that updates the state file to match real infrastructure's current attributes, without applying any changes from the configuration — useful for safely absorbing detected drift into state (accepting reality) without triggering a corrective apply.

Q: What is terraform import and where do you use it?
Ans: A command (`terraform import <resource_address> <real_resource_id>`) that brings an already-existing, unmanaged real resource under Terraform's management by adding it to the state file — used when adopting Terraform for infrastructure that was created manually/by another tool, without needing to recreate it.

Q: What is a null resource in Terraform?
Ans: A resource (`resource "null_resource" "example" {}"`) that doesn't correspond to any real infrastructure — used as a placeholder to run provisioners/trigger side effects (e.g., re-running a local script) based on `triggers`, when there's no natural resource to attach that logic to.

## Taint & Lifecycle
Q: What is Terraform Taint?
Ans: (Legacy: `terraform taint <resource>`) Manually marks a resource as needing to be destroyed and recreated on the next apply, even if its configuration hasn't changed — useful when a resource is known to be broken/corrupted but Terraform itself has no way to detect that. Modern Terraform prefers `terraform apply -replace=<resource_address>` instead.

Q: Terraform taint vs untaint?
Ans: `taint` marks a resource for forced recreation on next apply. `untaint` reverses that marking, removing the forced-recreation flag if you change your mind before applying.

Q: What is Untaint?
Ans: `terraform untaint <resource>` — removes a previously applied taint marker from a resource, so it will no longer be force-replaced on the next apply.

Q: What is a Terraform Lifecycle Block?
Ans: A nested `lifecycle { }` block within a resource that customizes how Terraform manages that specific resource's lifecycle — e.g., `create_before_destroy`, `prevent_destroy`, `ignore_changes`.

Q: What is prevent_destroy?
Ans: A lifecycle argument (`lifecycle { prevent_destroy = true }`) that causes Terraform to error out and refuse to destroy that specific resource, even if a plan would otherwise call for it — a safety guard for critical resources (e.g., a production database).

Q: Does prevent_destroy apply to the entire configuration?
Ans: No — it's scoped to the individual resource block it's set on. Other resources without it can still be destroyed normally; only the specific protected resource(s) are blocked.

Q: How do you delete a resource protected by prevent_destroy?
Ans: Remove (or set to `false`) the `prevent_destroy` lifecycle argument on that resource in the configuration first, apply that change, and only then run the `destroy`/removal — Terraform won't let you bypass it in a single step while it's still set to `true`.

## Workspaces & Modules
Q: What is Terraform Workspace?
Ans: A named, isolated instance of state for the same Terraform configuration — lets you manage multiple environments (dev/staging/prod) or parallel instances using the same `.tf` code, switched via `terraform workspace select <name>`.

Q: How do Terraform workspaces work?
Ans: Each workspace maintains its own separate state file (within the same configured backend), while sharing the same configuration code — `terraform workspace new/select/list/delete` manage them, and `terraform.workspace` is available as an interpolation inside the config (e.g., to vary a resource name per environment).

Q: Does each workspace have a separate state file?
Ans: Yes — each workspace's state is stored independently (e.g., under a different key/prefix in the same S3 bucket), so changes in one workspace never affect another's recorded state.

Q: What happens if you modify one workspace state file?
Ans: Only that workspace's resources/tracking are affected — other workspaces' state remains untouched since they're stored and managed completely independently, even though they share the same underlying `.tf` configuration.

Q: What are Terraform Modules?
Ans: Reusable, encapsulated groups of resources (a directory of `.tf` files) that can be called with input variables and expose output values — used to package common infrastructure patterns (e.g., a "VPC module") for reuse across multiple configurations/projects.

Q: How do you structure Terraform modules?
Ans: Typically: a root module (calls child modules, defines providers/backend), and child modules each in their own directory with a clear `variables.tf` (inputs), `main.tf` (resources), and `outputs.tf` (exposed values) — organized by logical infrastructure boundary (network, compute, database) so each module has a single, well-defined responsibility.

Q: How does a Terraform module differ from a template?
Ans: A module is a fully functional, composable unit of actual Terraform resource logic with typed inputs/outputs, versioning, and reuse across configurations. A "template" (e.g., a Terraform template file used with `templatefile()`) is just a static text/config file with variable placeholders rendered at plan/apply time — much simpler and not a reusable infrastructure component in itself.

## Resources & Expressions
Q: What is a Terraform Data Block?
Ans: A `data "provider_type" "name" { }` block that reads/queries information about an existing resource (managed by Terraform or not) without managing its lifecycle — used to reference values from resources created outside the current configuration.

Q: What is depends_on?
Ans: An explicit dependency declaration (`depends_on = [aws_instance.example]`) forcing Terraform to create/update/destroy resources in a specific order, used when a dependency isn't automatically inferable from resource attribute references alone (e.g., IAM eventual consistency, or side effects not expressed in the config).

Q: Difference between count and for_each?
Ans: `count` creates N near-identical copies of a resource, indexed numerically (`resource[0]`, `resource[1]`) — reordering the input list can cause unwanted resource recreation since indices shift. `for_each` creates one resource per key in a map/set, indexed by that stable key — safer for lists that may be reordered/modified, since each instance's identity is tied to its key, not its position.

Q: What are meta-arguments in Terraform?
Ans: Special arguments usable on any resource/module block that aren't provider-specific: `count`, `for_each`, `provider`, `depends_on`, and `lifecycle`.

Q: What is indexing in Terraform (e.g., count.index)?
Ans: `count.index` gives the current iteration number (0-based) when using the `count` meta-argument, letting you generate per-instance unique values (e.g., `name = "web-${count.index}"`, or picking a different AZ per index from a list).

Q: How do you create multiple EC2 instances with different configurations?
Ans: Use `for_each` over a map keyed by instance name, with each map value holding that instance's specific config (AMI, instance type, subnet, etc.), referencing `each.key`/`each.value` inside the resource block — this handles differing configurations far more cleanly than `count` (which assumes near-identical resources).

## Provisioners
Q: What are Terraform Provisioners?
Ans: Blocks (`local-exec`, `remote-exec`, `file`) that run scripts/commands on a local machine or a remote resource as part of resource creation/destruction — a last-resort escape hatch for actions Terraform's declarative resource model can't otherwise express.

Q: Types of Terraform Provisioners?
Ans: `local-exec` (runs a command on the machine running Terraform), `remote-exec` (runs commands on the newly-created remote resource via SSH/WinRM), and `file` (copies files/directories to the remote resource).

Q: Difference between Provider and Provisioner?
Ans: A **Provider** is the plugin implementing an entire platform's API (AWS, Azure) — it's how Terraform manages resources declaratively. A **Provisioner** is a narrow escape-hatch mechanism to run imperative scripts/commands during a resource's creation or destruction — used sparingly, since it breaks Terraform's declarative model and isn't tracked/reconciled like a normal resource.

Q: Why do we use Provisioners?
Ans: Only when something truly can't be expressed declaratively via a resource/provider — e.g., running a one-off bootstrap script, copying a file that a cloud-init/user-data mechanism can't handle. HashiCorp recommends avoiding them where possible in favor of native provider features (user_data, cloud-init) or dedicated config management tools (Ansible) instead.

## Secrets & Security
Q: How do you pass secrets to Terraform?
Ans: Via environment variables (`TF_VAR_xxx`), a `.tfvars` file excluded from version control, or — better — a dedicated secrets manager/data source (AWS Secrets Manager, Vault provider) queried at plan/apply time, so secrets are never hardcoded or committed in the `.tf` files themselves. Note that secret values still end up in the state file in plaintext unless the backend itself encrypts state at rest.

Q: How do you implement autoscaling using Terraform?
Ans: Define an `aws_launch_template`, an `aws_autoscaling_group` referencing it with min/max/desired capacity, and one or more `aws_autoscaling_policy` resources (target tracking, step scaling) tied to a CloudWatch alarm/metric — all declared as standard Terraform resources.

Q: How do you secure Terraform execution from Jenkins?
Ans: Grant the Jenkins agent a scoped IAM role (via instance profile or short-lived assumed-role credentials, never static keys) with least-privilege permissions for exactly the resources it manages, store state remotely with encryption + locking, require plan review/approval before apply on production, and avoid printing plan/apply output that might contain secret values in plaintext console logs.

## Rollback & Collaboration
Q: How do you handle Terraform rollback?
Ans: Terraform has no native "undo" — rollback means reverting the `.tf` configuration (via git revert) to the last known-good version and re-applying it, so infrastructure converges back to that prior declared state; keeping infra changes in small, frequently-applied commits makes this much less risky.

Q: How do multiple developers work on the same Terraform codebase?
Ans: Remote state with locking (S3+DynamoDB or Terraform Cloud) prevents concurrent apply conflicts; changes go through PRs with `terraform plan` output reviewed before merge; CI runs `plan` automatically on PRs and `apply` only on merge to main (or via manual approval); and modules/workspaces separate concerns so unrelated changes don't collide.

Q: How do you manage dependencies in Terraform?
Ans: Mostly implicitly — referencing one resource's attribute in another automatically creates a dependency edge in Terraform's graph, determining apply order. Explicit `depends_on` is used only when a dependency isn't inferable from attribute references. Cross-configuration dependencies (e.g., between separate Terraform projects/state files) are handled via `terraform_remote_state` data sources or well-defined outputs consumed as another config's input variables.

## Practical
Q: Write Terraform code to deploy an EC2 instance.
Ans:
```hcl
resource "aws_instance" "app_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```

## Troubleshooting
Q: Have you seen terraform plan pass but apply fail?
Ans: Yes — common causes: a race/eventual-consistency issue (e.g., an IAM role not yet fully propagated when a dependent resource tries to use it), a provider-side transient API error/throttling, a resource attribute only validated server-side at creation time (not caught by `plan`'s client-side checks), or concurrent external changes to the same resource between plan and apply.

Q: Have you seen terraform apply succeed but resources fail afterward?
Ans: Yes — e.g., an EC2 instance launches successfully (Terraform considers the `apply` complete once the API call succeeds and the resource exists) but the application inside it fails to start due to a bad user-data script, missing dependency, or misconfigured health check — Terraform's success only reflects that the *infrastructure* was created, not that the *workload* running on it is healthy.

## Hands-on Exercises

### Exercise 1: Launch an EC2 Instance
**Objective:** Write and apply Terraform configuration to launch an EC2 instance.
**Steps:**
1. Write a Terraform configuration for an EC2 instance.
2. Run the commands to apply it and create the instance.
3. Run `terraform apply` again — what happens?
4. Destroy the instance with Terraform.

**Solution:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "app_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```
```
terraform init
terraform validate
terraform plan      # Plan: 1 to add, 0 to change, 0 to destroy
terraform apply -auto-approve

# Running apply again: Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
# Terraform compares actual infra to config and finds no drift.

terraform destroy -auto-approve
```

### Exercise 2: Create a Custom VPC with Subnets
**Objective:** Create a VPC with two subnets across different Availability Zones.
**Steps:**
1. Create a VPC with CIDR block `10.0.0.0/16`.
2. Create two subnets, e.g. `10.0.0.0/20` and `10.0.16.0/20`, each in a different AZ.
3. Apply the configuration and confirm both resources are tracked in state.

**Solution:**
```hcl
resource "aws_vpc" "my_custom_vpc" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "my_custom_vpc" }
}

resource "aws_subnet" "subnet_a" {
  vpc_id            = aws_vpc.my_custom_vpc.id
  cidr_block        = "10.0.0.0/20"
  availability_zone = "us-west-2a"
  tags = { Name = "Subnet A" }
}

resource "aws_subnet" "subnet_b" {
  vpc_id            = aws_vpc.my_custom_vpc.id
  cidr_block        = "10.0.16.0/20"
  availability_zone = "us-west-2b"
  tags = { Name = "Subnet B" }
}
```
```
terraform init
terraform apply -auto-approve
```
