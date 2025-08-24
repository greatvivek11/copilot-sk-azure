# Azure Best Practices Guide

## Philosophy

* Design with the **Azure Well-Architected Framework**: Cost, Reliability, Security, Performance, Operational Excellence.
* Favor **managed services** over VMs wherever possible.
* **Automate everything**: IaC (Bicep/Terraform), CI/CD, monitoring.
* Use **least privilege principle** and enforce **Zero Trust** networking.
* Always design for **resilience & scalability** (multi-region if critical).

## Identity & Access (IAM)

* Use **Azure AD (Entra ID)** for authentication everywhere.
* Prefer **Managed Identities** over client secrets.
* Assign RBAC at the **lowest scope** possible.
* Use **PIM (Privileged Identity Management)** for admin roles.
* Regularly review **access logs and role assignments**.

## Networking & Security

* Default deny: block public endpoints unless required.
* Use **Private Endpoints** to connect services (Storage, SQL, Cosmos).
* Secure traffic with **NSGs & ASGs**.
* Enable **DDoS protection** for internet-facing workloads.
* Enforce **TLS 1.2+**.
* Use **Key Vault** for secrets, keys, certs.

## Resource Management (ARM, Bicep, Terraform)

* Use **Bicep** (preferred ARM DSL) or Terraform for IaC.
* Enforce **naming conventions** and **tags** (env, owner, cost center).
* Use **Resource Groups by lifecycle**, not by service type.
* Apply **Azure Policy** for guardrails (SKU restrictions, region restrictions).
* Enable **Blue/Green or Canary deployments** via deployment slots.

## Compute & Containers

* **App Service**: Use slots, autoscaling, MI for DB access.
* **Azure Functions**: Prefer Consumption or Premium plan; design for idempotency.
* **Azure Container Apps**: Great for microservices; use scale-to-zero for cost savings.
* **AKS**: Use only when container orchestration is needed; otherwise ACA/App Service is simpler.
* Avoid putting business logic in VMs unless absolutely required.

## Data & Storage

* **SQL Database**: Use Managed Identity, enable Auditing, Threat Detection.
* **Cosmos DB**: Choose partition keys carefully; enable autoscale throughput.
* **Storage Account**: Disable public blob access; enforce private endpoints.
* Enable **Geo-redundant storage (GRS)** for critical workloads.
* Use **Soft Delete & Versioning** for Blob and Table storage.

## Observability & Monitoring

* Use **Azure Monitor + App Insights** for distributed tracing.
* Enable **Log Analytics workspace** for centralized logging.
* Configure **alerts** for anomalies, budget thresholds, and service health.
* Use **Dashboarding** for SRE/DevOps visibility.
* Enable **diagnostic settings** to route logs to Log Analytics/Event Hub/Storage.

## Serverless & Messaging

* **Azure Functions**: Use Durable Functions for orchestrations.
* **Logic Apps**: Use for integrations and workflows (B2B, SaaS connectors).
* **Service Bus**: Reliable messaging with queues, topics, DLQ.
* **Event Grid**: Reactive, event-driven systems at scale.
* **Event Hubs**: For telemetry & streaming ingestion.

## Governance & Cost Management

* Use **Management Groups** to enforce policies org-wide.
* Apply **Budgets + Alerts** in Cost Management.
* Right-size resources with **Advisor recommendations**.
* Deallocate unused VMs; scale down idle services.
* Review **spending reports** monthly with FinOps practice.

## Anti-Patterns

* Hardcoding secrets in code (always use **Key Vault**).
* Overusing **Key Vault calls** in hot paths (cache instead).
* Exposing DB/Storage with public endpoints.
* Granting Contributor/Owner instead of granular RBAC.
* Relying only on **default VNET** (use Hub-Spoke architecture).
* Ignoring **N+1 query issues** in EF Core on Azure SQL.
* Using **Functions with long-running sync code** (use Durable Functions).
* Deploying to a single region without DR/backup strategy.
