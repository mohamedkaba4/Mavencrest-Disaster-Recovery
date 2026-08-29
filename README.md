# Mavencrest Disaster Recovery

Enterprise-style disaster recovery automation for the Mavencrest Azure container environment using Azure DevOps, Terraform, Azure CLI, and automated recovery decision options.

This project demonstrates how infrastructure recovery can be orchestrated safely after partial outages, complete infrastructure loss, or Terraform state-related failures. Rather than immediately running Terraform to recreate resources, the  pipeline first determines the current state of the environment and selects the safest recovery path.

Preflight 
     ↓
Recovery Decision 
     ↓ 
Recovery Process 
     ↓ 
Validation

## Recovery Modes
auto — Detect the current failure condition and choose a recovery strategy (healthy,
partial-recovery, full-rebuild, or state-recovery-required)
full-rebuild — Rebuild the production infrastructure
validate-only — Run recovery checks without making changes
This repository handles a full recovery using two repositories as its source of truth:

- `MavencrestAzure` — Terraform IaC
- `E-commerce` — application source and Docker images

## Recovery scenarios

### Reconcile - Drift detection
Terraform state and production infrastructure both exist.

The platform can be reconciled through Terraform without rebuilding the environment manually.

### State Loss -  incomplete environment
When production resources exist, but Terraform state is not available.

The pipeline must recover state or import part of the existing infrastructure. It should not blindly recreate production resources.

### Full Rebuild
Production infrastructure or the platform is completly unavailable.

The DR workflow rebuilds:

1. Terraform backend
2. Azure foundation
3. Application container images
4. Azure workloads
5. Production health checks

## Archetecture for Recovery Classification

The DR pipeline first inspects the production environment and Terraform state backend before taking recovery action.

```text
AUTO classification
│
├─ prod + state exist
│    → reconcile
│    → DR pipeline does NOT rebuild
│    → use normal orchestration
│
├─ prod exists + state missing
│    → state-loss
│    → STOP
│    → recover state first
│
└─ prod missing
     → full-rebuild
          ↓
     Resolve/Create backend
          ↓
     Foundation PLAN
          ↓
     HUMAN APPROVAL
          ↓
     Foundation APPLY
          ↓
     Build app images
          ↓
     Workload PLAN
          ↓
     HUMAN APPROVAL
          ↓
     Workload APPLY
          ↓
     Health checks
```

##Safety Features
Manual-only DR execution
Terraform backend/state checks
Azure resource detection
State-loss protection
Planned destructive Terraform plan detection
Post-recovery validation

**Note:** The current automatic recovery classifier uses the Terraform
> backend and production resource group as its primary signals.
> Resource-level health and partial-outage detection will be added in a later update.

> An expansion to the project would also include:
  cloud-based disaster recovery automated backup policies, 
  cross-region replication,
  rapid failover to improve business continuity while minimizing downtime.
