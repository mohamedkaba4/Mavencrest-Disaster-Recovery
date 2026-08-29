# Mavencrest Disaster Recovery

Disaster recovery orchestration for the Mavencrest Azure container platform.

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
## Safety

The DR pipeline is:

- manually triggered
- scenario-aware based on recovry scenerios
- designed to separate plan from apply
- protected from automatically rebuilding resources during state-loss scenarios or other partially incomplete cases
- intended to use Azure DevOps production approvals before destructive recovery actions

**Note:** The current automatic recovery classifier uses the Terraform
> backend and production resource group as its primary signals.
> Resource-level health and partial-outage detection will be added in a later update.

> An expensansion to the project would also include:
  cloud-based disaster recovery automated backup policies, 
  cross-region replication,
  rapid failover to improve business continuity while minimizing downtime.
