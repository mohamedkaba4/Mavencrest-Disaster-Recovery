# Mavencrest Disaster Recovery

Disaster recovery orchestration for the Mavencrest Azure container platform.

This repository handles a full recovery using two authoritative source repositories:

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

## Safety

The DR pipeline is:

- manually triggered
- scenario-aware based on recovry scenerios
- designed to separate plan from apply
- protected from automatically rebuilding resources during state-loss scenarios or other partially incomplete cases
- intended to use Azure DevOps production approvals before destructive recovery actions
