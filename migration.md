# Infrastructure Migration Proposal

## Azure AKS → Vultr Kubernetes

**Prepared for:** Management & Project Stakeholders

---

# Executive Summary

This proposal outlines the migration of our production infrastructure from **Microsoft Azure AKS** to **Vultr Kubernetes Infrastructure**.

The primary objective is to significantly reduce recurring infrastructure costs while maintaining the current production architecture, reliability, and scalability.

Based on current estimates, the migration is expected to reduce infrastructure spending by **approximately 55%**, lowering monthly operational costs from **approximately $450/month** to **approximately $200/month**.

---

# Migration Objectives

* Reduce recurring cloud infrastructure costs
* Maintain production availability and reliability
* Preserve existing Kubernetes-based deployment architecture
* Improve cost predictability through fixed compute pricing
* Reduce dependence on Azure-managed infrastructure

---

# Proposed Infrastructure

## Networking

| Component      | Quantity |
| -------------- | -------: |
| Load Balancers |        2 |

The load balancers will provide:

* External traffic routing
* High availability
* Ingress for Kubernetes services

---

## Compute Infrastructure

Five dedicated compute instances will host the platform services.

| Service                                      | Specification                               | Monthly Cost |
| -------------------------------------------- | ------------------------------------------- | -----------: |
| Admin Service                                | 2 vCPU, 4 GB RAM, 100 GB SSD, 5 TB Transfer |      **$24** |
| Dashboard Service                            | 2 vCPU, 4 GB RAM, 100 GB SSD, 5 TB Transfer |      **$24** |
| PostgreSQL Database                          | 2 vCPU, 4 GB RAM, 100 GB SSD, 5 TB Transfer |      **$24** |
| Worker Service                               | 2 vCPU, 4 GB RAM, 100 GB SSD, 5 TB Transfer |      **$24** |
| Monitoring Stack (Prometheus, Grafana, Loki) | 2 vCPU, 4 GB RAM, 100 GB SSD, 5 TB Transfer |      **$24** |

**Total Compute Cost:** **$120/month**

---

# Supporting Services

| Service            | Purpose                    | Monthly Cost |
| ------------------ | -------------------------- | -----------: |
| Load Balancers (2) | External traffic routing   |      **$20** |
| Docker Hub Team    | Private container registry |      **$16** |
| npm Team Plan      | Private package registry   |       **$7** |

---

# Variable Services

The following services are billed based on actual usage.

| Service            | Purpose                                      | Estimated Monthly Cost |
| ------------------ | -------------------------------------------- | ---------------------: |
| Azure Cosmos DB    | Application database                         |             **$15–30** |
| Azure Blob Storage | Uploaded files, logs, and application assets |              **$5–15** |
| Object Storage     | Database backups and long-term storage       |              **$5–10** |

---

# Estimated Monthly Infrastructure Cost

| Category               | Estimated Cost |
| ---------------------- | -------------: |
| Compute Infrastructure |       **$120** |
| Load Balancers         |        **$20** |
| Docker Hub Team        |        **$16** |
| npm Team Plan          |         **$7** |
| Azure Cosmos DB        |     **$15–30** |
| Azure Blob Storage     |      **$5–15** |
| Backup Object Storage  |      **$5–10** |

---

# Total Estimated Monthly Cost

| Scenario           | Monthly Cost |
| ------------------ | -----------: |
| Minimum Expected   |     **$188** |
| Recommended Budget |    **~$200** |
| Higher Usage       |     **$218** |

For budgeting purposes, allocating **$200/month** is recommended.

---

# Cost Comparison

| Infrastructure                | Monthly Cost |
| ----------------------------- | -----------: |
| Current Azure AKS             |    **~$450** |
| Proposed Vultr Infrastructure |    **~$200** |

---

# Estimated Savings

| Metric          |                 Value |
| --------------- | --------------------: |
| Monthly Savings |             **~$250** |
| Annual Savings  |           **~$3,000** |
| Cost Reduction  | **Approximately 55%** |

---

# Infrastructure Components

## Application Layer

* Admin Service
* Dashboard Service
* Worker Service

## Data Layer

* PostgreSQL
* Azure Cosmos DB

## Storage

* Azure Blob Storage
* Object Storage (Backups)

## Monitoring

* Prometheus
* Grafana
* Loki

## Registries

* Docker Hub Team
* npm Team Registry

---

# Assumptions

* Each compute instance includes:

  * 2 vCPUs
  * 4 GB RAM
  * 100 GB SSD
  * 5 TB monthly bandwidth
* Cosmos DB, Azure Blob Storage, and backup object storage costs are estimated based on expected production usage and may vary.
* Monitoring services are self-hosted on the dedicated monitoring instance.
* SSL certificates will use Let's Encrypt.
* The proposed infrastructure is sized for the current production workload and can be scaled as demand increases.

---

# Risks and Considerations

* Variable service costs (Cosmos DB and storage) depend on actual usage.
* Temporary dual-running costs may occur during the migration period.
* Bandwidth charges may increase if transfer exceeds the included limits.
* Routine VM snapshots and backups should be configured for disaster recovery.

---

# Expected Benefits

* Approximately **55% reduction** in monthly infrastructure costs.
* Estimated **$3,000 annual savings**.
* Predictable infrastructure pricing.
* Continued use of Kubernetes without major architectural changes.
* Easier infrastructure management and scaling.
* Reduced dependence on Azure-specific managed services.

---

# Recommendation

Proceed with the migration from **Azure AKS** to **Vultr Kubernetes**.

The proposed infrastructure meets the current production requirements while reducing recurring cloud expenditure from approximately **$450/month** to **$200/month**, resulting in an estimated **$250/month** (**~55%**) cost reduction without requiring significant application changes.
