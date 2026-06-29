# Terraform IaC — Referenzmodul

Projekt-Struktur, Module, State-Management, CI/CD-Integration, Sicherheit, Testing.

---

## Empfohlene Projektstruktur

```
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf          # Modul-Aufrufe
│   │   ├── variables.tf     # Env-spezifische Variablen
│   │   ├── outputs.tf
│   │   ├── backend.tf       # Remote State Konfiguration
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
├── modules/
│   ├── networking/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md        # terraform-docs generiert
│   ├── aks-cluster/
│   ├── sql-server/
│   └── monitoring/
├── .terraform-version        # tfenv Pin
├── .tflint.hcl
└── .pre-commit-config.yaml
```

---

## Azure Backend + Provider (produktionsreif)

```hcl
# environments/prod/backend.tf
terraform {
  required_version = ">= 1.7.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.100"
    }
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.48"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.6"
    }
  }

  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformfirmaprod"
    container_name       = "tfstate"
    key                  = "prod/terraform.tfstate"
    use_oidc             = true   # Keyless Auth via GitHub OIDC
  }
}

provider "azurerm" {
  features {
    resource_group { prevent_deletion_if_contains_resources = true }
    key_vault {
      purge_soft_delete_on_destroy    = false
      recover_soft_deleted_key_vaults = true
    }
  }
  use_oidc = true
}

locals {
  env  = var.environment
  proj = var.project
  common_tags = {
    Environment = var.environment
    Project     = var.project
    ManagedBy   = "Terraform"
    Repository  = "github.com/firma/infra"
  }
}
```

---

## Modul-Beispiel: AKS-Cluster

```hcl
# modules/aks-cluster/main.tf
resource "azurerm_kubernetes_cluster" "main" {
  name                = "aks-${var.project}-${var.environment}-${var.location_short}"
  location            = var.location
  resource_group_name = var.resource_group_name
  dns_prefix          = "${var.project}-${var.environment}"
  kubernetes_version  = var.kubernetes_version
  sku_tier            = var.environment == "prod" ? "Standard" : "Free"

  default_node_pool {
    name                = "system"
    node_count          = var.system_node_count
    vm_size             = var.system_node_size
    os_disk_type        = "Ephemeral"
    type                = "VirtualMachineScaleSets"
    vnet_subnet_id      = var.subnet_id
    only_critical_addons_enabled = true

    upgrade_settings {
      max_surge = "33%"
    }
  }

  identity {
    type         = "UserAssigned"
    identity_ids = [azurerm_user_assigned_identity.aks.id]
  }

  network_profile {
    network_plugin    = "azure"
    network_policy    = "azure"
    load_balancer_sku = "standard"
    outbound_type     = "userDefinedRouting"
  }

  azure_active_directory_role_based_access_control {
    managed            = true
    azure_rbac_enabled = true
    tenant_id          = data.azurerm_client_config.current.tenant_id
  }

  oms_agent {
    log_analytics_workspace_id      = var.log_analytics_workspace_id
    msi_auth_for_monitoring_enabled = true
  }

  microsoft_defender {
    log_analytics_workspace_id = var.log_analytics_workspace_id
  }

  maintenance_window_auto_upgrade {
    frequency   = "Weekly"
    interval    = 1
    day_of_week = "Sunday"
    start_time  = "02:00"
    utc_offset  = "+01:00"
    duration    = 4
  }

  tags = var.tags

  lifecycle {
    ignore_changes = [default_node_pool[0].node_count]
    prevent_destroy = true
  }
}

# User Node Pool (Workloads)
resource "azurerm_kubernetes_cluster_node_pool" "workload" {
  name                  = "workload"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.main.id
  vm_size               = var.workload_node_size
  min_count             = var.workload_min_count
  max_count             = var.workload_max_count
  enable_auto_scaling   = true
  vnet_subnet_id        = var.subnet_id
  os_disk_type          = "Ephemeral"
  node_labels           = { "workload" = "true" }
  tags                  = var.tags
}
```

---

## Terraform CI/CD (GitHub Actions)

```yaml
name: Terraform
on:
  push:
    branches: [main]
    paths: ['terraform/**']
  pull_request:
    paths: ['terraform/**']

permissions:
  id-token: write
  contents: read
  pull-requests: write

env:
  TF_VERSION: '1.7.5'
  WORKING_DIR: terraform/environments/prod

jobs:
  validate:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ env.WORKING_DIR }}
    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - uses: actions/cache@v4
        with:
          path: ~/.terraform.d/plugin-cache
          key: tf-${{ hashFiles('**/.terraform.lock.hcl') }}

      - name: Azure Login (OIDC)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - run: terraform init
      - run: terraform validate
      - run: terraform fmt -check -recursive

      - name: tflint
        uses: terraform-linters/setup-tflint@v4
      - run: tflint --init && tflint --recursive

      - name: Checkov (Security Scan)
        uses: bridgecrewio/checkov-action@master
        with:
          directory: ${{ env.WORKING_DIR }}
          framework: terraform
          soft_fail: false
          skip_check: CKV_AZURE_131  # Bekannte Ausnahme, dokumentiert

  plan:
    needs: validate
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ env.WORKING_DIR }}
    outputs:
      plan-exitcode: ${{ steps.plan.outputs.exitcode }}
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - run: terraform init
      - name: Terraform Plan
        id: plan
        run: |
          terraform plan -out=tfplan -detailed-exitcode 2>&1 | tee plan.txt
          echo "exitcode=${PIPESTATUS[0]}" >> $GITHUB_OUTPUT
      - uses: actions/upload-artifact@v4
        with:
          name: tfplan
          path: ${{ env.WORKING_DIR }}/tfplan

      # Plan-Output als PR-Kommentar
      - uses: actions/github-script@v7
        if: github.event_name == 'pull_request'
        with:
          script: |
            const fs = require('fs');
            const plan = fs.readFileSync('${{ env.WORKING_DIR }}/plan.txt', 'utf8');
            const truncated = plan.length > 60000 ? plan.slice(0,60000) + '\n...(truncated)' : plan;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '## Terraform Plan\n```hcl\n' + truncated + '\n```'
            });

  apply:
    needs: plan
    if: github.ref == 'refs/heads/main' && needs.plan.outputs.plan-exitcode == '2'
    runs-on: ubuntu-latest
    environment: production
    defaults:
      run:
        working-directory: ${{ env.WORKING_DIR }}
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - uses: actions/download-artifact@v4
        with:
          name: tfplan
          path: ${{ env.WORKING_DIR }}
      - run: terraform init
      - run: terraform apply -auto-approve tfplan
```

---

## Secrets Management — Kein Hardcoding

```hcl
# Azure Key Vault Integration
data "azurerm_key_vault_secret" "db_password" {
  name         = "db-password-prod"
  key_vault_id = azurerm_key_vault.main.id
}

resource "azurerm_mssql_server" "main" {
  administrator_login_password = data.azurerm_key_vault_secret.db_password.value
  # ...
}

# Sensible Outputs maskieren
output "connection_string" {
  value     = "Server=${azurerm_mssql_server.main.fully_qualified_domain_name};..."
  sensitive = true
}
```

---

## Terraform Testing (terraform test, terratest)

```hcl
# tests/networking.tftest.hcl (natives terraform test Framework ab v1.6)
run "validates_vnet_cidr" {
  command = plan

  variables {
    vnet_cidr = "10.0.0.0/16"
    environment = "test"
  }

  assert {
    condition     = azurerm_virtual_network.main.address_space[0] == "10.0.0.0/16"
    error_message = "VNet CIDR stimmt nicht überein."
  }
}

run "creates_subnets" {
  command = apply

  assert {
    condition     = length(azurerm_subnet.main) >= 3
    error_message = "Mindestens 3 Subnetze erwartet."
  }
}
```

```go
// terratest — Integration Test (Go)
func TestAKSCluster(t *testing.T) {
    opts := &terraform.Options{
        TerraformDir: "../modules/aks-cluster",
        Vars: map[string]interface{}{
            "environment": "test",
            "project":     "unittest",
        },
    }
    defer terraform.Destroy(t, opts)
    terraform.InitAndApply(t, opts)

    clusterName := terraform.Output(t, opts, "cluster_name")
    assert.Contains(t, clusterName, "unittest")
}
```
