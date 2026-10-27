# Terraform Notes

## Links

* [https://developer.hashicorp.com/terraform/docs](https://developer.hashicorp.com/terraform/docs)

A practical refresher using [tf-shared-core-services](https://github.com/burrt/tf-shared-core-services) and its connection to [AwsLambdaDotnetWebApi](https://github.com/burrt/AwsLambdaDotnetWebApi). The economic-data fetcher provides familiar names for configuration examples.

### Configuration basics

Reference: [Configuration language](https://developer.hashicorp.com/terraform/language)

* **Configuration:** desired infrastructure. **Provider:** API integration. **State:** recorded object identities. **Plan:** proposed actions. **Apply:** execute changes.
* All `.tf` files in one directory form one module. Filenames do not determine execution order.
* Subdirectories are not loaded automatically; include them using a `module` block.

| Block      | Purpose                     | Reference example                 |
| ---------- | --------------------------- | --------------------------------- |
| `provider` | Configure an integration    | AWS region / credentials          |
| `variable` | Declare an input            | `var.environment`                 |
| `locals`   | Name an internal expression | `local.redis_name`                |
| `resource` | Manage an object            | `aws_vpc.shared.id`               |
| `data`     | Read existing information   | `data.aws_vpc.existing.id`        |
| `module`   | Include child configuration | `module.network.vpc_id`           |
| `output`   | Expose a result             | Child output or saved root output |

```hcl
resource "aws_vpc" "shared" {
  cidr_block = "10.0.0.0/16"
  tags       = { Name = "shared-vpc" }
}
# aws_vpc.shared    = Terraform address
# aws_vpc.shared.id = AWS ID, usually known after initial apply
# shared           = local label, not automatically an AWS Name tag
```

```hcl
# Alternative: look up an existing VPC, without creating one.
data "aws_vpc" "existing" {
  id = var.existing_vpc_id
}
# Reference: data.aws_vpc.existing.id
```

### Variables, tfvars, locals, and outputs

References: [Input variables](https://developer.hashicorp.com/terraform/language/values/variables) · [Locals](https://developer.hashicorp.com/terraform/language/values/locals)

| Mechanism        | Role                        | Example                     |
| ---------------- | --------------------------- | --------------------------- |
| `variable` block | Declare configurable input  | Environment / Redis count   |
| `.tfvars` file   | Supply root input values    | Production settings         |
| `locals` block   | Define internal expressions | Derived names / common tags |
| `output` block   | Expose a value              | VPC ID / subnet IDs         |

#### Declare inputs; supply values

```hcl
# variables.tf — declare inputs
variable "environment" {
  type    = string
  default = "dev"
}

variable "redis_node_count" {
  type    = number
  default = 1
}

variable "my_local_ip" {
  type = string # No default: caller must supply it.
}
```

```hcl
# prod.tfvars — supply values, not variable blocks
# This filename is NOT auto-loaded: use -var-file=prod.tfvars.
environment      = "prod"
redis_node_count = 3
my_local_ip      = "203.0.113.10/32" # Documentation-only CIDR.
# var.environment = "prod"; var.redis_node_count = 3
```

* Auto-loaded: `terraform.tfvars`, `terraform.tfvars.json`, `*.auto.tfvars`, `*.auto.tfvars.json`.
* Other filenames need `-var-file`. Environment variables use `TF_VAR_environment`, etc.
* `.tfvars` assigns root inputs, not locals or child inputs. It cannot reference `var.*` / `local.*` / resources or call functions.
* Ordinary CLI precedence, low → high: **default → TF\_VAR → terraform.tfvars → terraform.tfvars.json → auto files (lexical order) → -var/-var-file (argument order)**.
* Later assignments override earlier ones; maps are replaced, not automatically merged. HCP workspace/variable-set values have additional priority rules.

#### Locals: reusable expressions inside the module

```hcl
# Using environment="prod" from prod.tfvars:
locals {
  redis_name = "${var.environment}-shared-redis" # "prod-shared-redis"

  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }

  redis_tags = merge(local.common_tags, {
    Component = "redis"
    Name      = local.redis_name
  })
  # { Environment="prod", ManagedBy="Terraform",
  #   Component="redis", Name="prod-shared-redis" }
}
```

```hcl
# Example arguments inside a Redis resource:
replication_group_id = local.redis_name
num_cache_clusters   = var.redis_node_count
# Configure the remaining Redis/HA arguments consistently with node count.
```

* Declare **`locals`**, reference singular **`local.name`**.
* Use for shared tags, derived names, filtering, and reusable calculations.
* Locals can refer to variables, resources, and other locals. They are named expressions, not mutable Python variables.
* Scope: current module, across its `.tf` files. They do not mean "local computer".
* A local cannot be overridden through `.tfvars`; expose an input if callers need control.

#### Outputs: expose results

```hcl
output "redis_name" {
  value = local.redis_name # "prod-shared-redis"
}

output "vpc_id" {
  value = aws_vpc.shared.id # Existing/created AWS VPC ID.
}
# Child output: module.network.vpc_id
# Root outputs: saved with state and available through terraform output.
# Matching variable/output names do not connect them automatically.
```

### Types and expressions

References: [Types](https://developer.hashicorp.com/terraform/language/expressions/type-constraints) · [For expressions](https://developer.hashicorp.com/terraform/language/expressions/for)

| Type            | Meaning                                |
| --------------- | -------------------------------------- |
| `string`        | `"ap-southeast-2"`                     |
| `number`        | `3`                                    |
| `bool`          | `true`                                 |
| `list(string)`  | Ordered strings; duplicates allowed    |
| `set(string)`   | Unique strings; no positional indexing |
| `map(number)`   | String keys, one common value type     |
| `object({...})` | Named attributes with individual types |
| `tuple([...])`  | Positions with individual types        |

```hcl
variable "redis" {
  type = object({
    enabled    = bool
    node_count = number
  })
  default = { enabled = true, node_count = 3 }
}
# var.redis.enabled = true; var.redis.node_count = 3
```

* `[]` constructs a tuple, `{}` an object; compatible values convert to declared list/map types where required.
* `null` means absence; resource arguments generally treat it as omitted.
* Unknown means "determined later", not null. A new subnet ID can be unknown at plan time.

#### Functions and conditionals

```hcl
# Console expressions; simple results are comments.
"prod" == "prod" ? 3 : 1                        # 3
length(["10.0.10.0/24", "10.0.11.0/24"])         # 2
lookup({ bea = 120 }, "fred", 60)                # 60 (key absent)
merge({ Env = "dev" }, { Env = "prod" })         # { Env = "prod" }
toset(["bea", "bea", "fred"])                   # Set of "bea" and "fred"
flatten([["bea"], ["bls", "fred"]])              # ["bea", "bls", "fred"]
try({ redis = {} }.redis.node_type, "cache.t3.micro") # "cache.t3.micro"
try(null, "fallback")                           # null: no error occurred
```

* Conditional branches need compatible types.
* `merge` uses the later value for duplicate keys.
* `try` handles dynamic expression errors, not failed AWS operations or undeclared top-level references.

#### Comprehensions: transform and filter

```hcl
locals {
  redis_clusters = {
    shared     = { enabled = true, node_count = 3 }
    experiment = { enabled = false, node_count = 1 }
  }

  enabled_node_counts = {
    for name, cluster in local.redis_clusters :
    name => cluster.node_count
    if cluster.enabled
  } # { shared = 3 }
  # name = key; cluster = whole inner object
  # => separates new key/value; if excludes disabled entries

  provider_labels = [for source in ["bea", "fred"] : upper(source)]
  # ["BEA", "FRED"]
}
```

* **`for` builds values; `for_each` determines resource/module instances.**
* Braces produce an object; brackets produce a tuple.

### Dependencies, count, and for\_each

References: [count](https://developer.hashicorp.com/terraform/language/meta-arguments/count) · [for\_each](https://developer.hashicorp.com/terraform/language/meta-arguments/for_each)

* Referencing `aws_vpc.shared.id` in a subnet creates an implicit dependency on that VPC. File order is irrelevant.
* `depends_on` declares a hidden operational dependency that value references do not express.

#### count: indexed instances

```hcl
locals {
  private_cidrs = ["10.0.10.0/24", "10.0.11.0/24"]
}

resource "aws_subnet" "private" {
  count      = length(local.private_cidrs)
  vpc_id     = aws_vpc.shared.id
  cidr_block = local.private_cidrs[count.index]
}
# aws_subnet.private[0] → 10.0.10.0/24
# aws_subnet.private[1] → 10.0.11.0/24
# Remove first CIDR: [0] receives the second CIDR; [1] is removed.
# Update versus replacement depends on the provider/resource schema.
```

```hcl
# Inside an optional jump-box resource:
count = var.enable_jump_box ? 1 : 0
# false → no instances; true → one instance addressed with [0]
```

#### for\_each: instances identified by keys

```hcl
locals {
  private_cidrs = {
    private_1 = "10.0.10.0/24"
    private_2 = "10.0.11.0/24"
  }
}

resource "aws_subnet" "private" {
  for_each   = local.private_cidrs
  vpc_id     = aws_vpc.shared.id
  cidr_block = each.value
  tags       = { Name = "shared-${each.key}" }
}
# aws_subnet.private["private_1"] → 10.0.10.0/24
# aws_subnet.private["private_2"] → 10.0.11.0/24
# Remove private_1: private_2 retains its address.
```

* Accepts a **map or set of strings**, not an unconverted list.
* Map: `each.key` is its key; `each.value` its value. String set: both are the same string.
* `toset(["bea", "bea", "fred"])` gives two instance keys.
* Ordinary planning needs a known count or known, nonsensitive `for_each` keys; other attributes can remain unknown.
* One block cannot use both mechanisms. Renaming a key changes identity.
* `dynamic` repeats nested blocks, such as security-group ingress rules; it does not create separate resource instances.

### Modules and separate deployments

Reference: [Module blocks](https://developer.hashicorp.com/terraform/language/block/module)

```hcl
# Child: modules/shared-network/main.tf
variable "cidr" {
  type = string
}

resource "aws_vpc" "shared" {
  cidr_block = var.cidr
}

output "vpc_id" {
  value = aws_vpc.shared.id
}
```

```hcl
# Parent/root module:
module "network" {
  source = "./modules/shared-network"
  cidr   = "10.0.0.0/16" # Explicit child input.
}

output "vpc_id" {
  value = module.network.vpc_id # Child's exposed output.
}
# Internal address: module.network.aws_vpc.shared
# Access output via module.network.vpc_id, not the internal address above.
```

* Child modules do not automatically inherit parent variables or locals.
* Default provider configurations can be inherited; required provider aliases must be passed explicitly.
* Registry module versions use `version`; Git sources use `?ref=...`. `//` selects a Git source subdirectory.
* **Child module:** includes code in this deployment. **Remote state:** reads outputs from a separate deployment.
* Your Lambda can read `data.terraform_remote_state.network.outputs.subnet_private_1_id` to use an existing shared subnet. That does not create another subnet or apply the shared workspace first.
* Remote-state access needs permission to read the underlying snapshot. Public GitHub code does not grant workspace-state access.

### State and lifecycle

Reference: [Lifecycle behaviour](https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle)

* **State:** maps addresses such as `aws_vpc.shared` to real object IDs.
* **Drift:** infrastructure changed outside Terraform. A normal plan refreshes managed-object information and may propose reconciling it with configuration.
* Removing a resource block normally proposes destruction. Deleting state alone does not delete the real infrastructure.
* **Import:** associate an existing object with a Terraform address; it does not clone the object.
* Backend locking, where supported, prevents conflicting state operations.

```hcl
moved {
  from = aws_vpc.main-vpc
  to   = aws_vpc.shared
}
# Also rename the resource declaration: records the address move.
# Other argument changes can still cause replacement.
# Moving a block between .tf files in the same module changes no address.
```

Lifecycle options inside a resource:

* **`create_before_destroy = true`:** create replacement before destroying the old object. Temporary coexistence must be possible.
* **`prevent_destroy = true`:** reject destruction while protection remains configured. Deleting the whole resource block removes this protection too.
* **`ignore_changes = [tags]`:** use tags on creation, ignore their differences during updates; other attributes remain managed.
* A Lambda's `ignore_changes = [source_code_hash]` ignores that attribute during updates, not the whole Lambda.

### Plans, versions, and sensitive values

References: [CLI commands](https://developer.hashicorp.com/terraform/cli/commands) · [Version constraints](https://developer.hashicorp.com/terraform/language/expressions/version-constraints)

Only the command distinctions worth remembering for the assessment:

* **`init`:** initialize backend, providers, modules. **`validate`:** configuration/schema checks, not proof deployment succeeds.
* **`plan`:** preview changes. **`apply`:** execute changes. A saved plan can be applied directly; `apply -auto-approve` alone creates a new plan and skips its approval.
* Plan symbols: **`+` create**, **`~` update**, **`-` destroy**, **`-/+` destroy/create replacement**, **`+/-` create/destroy replacement**.
* `(known after apply)` means a value will become available later.

```hcl
terraform {
  required_version = ">= 1.9.8" # Terraform CLI constraint.
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.76" # >= 5.76.0 and < 6.0.0; example constraint.
    }
  }
}
# "~> 5.76.0" → >= 5.76.0 and < 5.77.0
# "= 5.76.0"  → exactly 5.76.0
```

* `.terraform.lock.hcl` records provider selections/checksums; normally commit it. It does not pin Git module revisions.
* Ordinary `init` reuses suitable locked selections; `init -upgrade` reconsiders versions within constraints.

```hcl
variable "redis_password" {
  type      = string
  sensitive = true
}
# Redacted in normal display; ordinary values can still be stored in state/plan.
```

* `sensitive` is not encryption or automatic omission from state.
* Newer exception: **ephemeral values (1.10+)** and **provider-supported write-only arguments (1.11+)** can omit values from saved state/plan, with usage restrictions. Reading a secret from a secret store alone does not do this.
