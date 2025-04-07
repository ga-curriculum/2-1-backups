<h1>
  <span class="headline">3-2-1 Backup Strategy</span>
  <span class="subhead">Implementation Architectures</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to design backup architectures that implement the 3-2-1 strategy across different technological environments while meeting specified RTO and RPO requirements.

## Implementing 3-2-1 in Modern Environments

Today's IT environments are rarely simple. They span on-premises infrastructure, multiple cloud providers, SaaS applications, and edge computing. Let's explore how to implement 3-2-1 backup strategies across these complex landscapes.

### Hybrid Cloud Implementation Example

**TODO: Add Diagram (Ask David What that Should look like)**

**Components:**

1. **Production Environment**

- VMware-based virtualized workloads
- Application-consistent snapshots via VMware APIs

2. **Local Backup Infrastructure**

- Deduplication to reduce storage requirements
- Instant recovery capabilities for critical VMs
- **RPO:** 1 hour, **RTO:** 15 minutes for tier-1 applications

3. **Cloud Backup Target**

- Encrypted backup sets sent to cloud storage
- Immutable storage with versioning enabled
- Geographic redundancy within cloud provider
- **RPO:** 24 hours, **RTO:** 4 hours for full site recovery

### Cross-Cloud Replication Architecture

Enterprises increasingly use multiple clouds for redundancy. This approach, sometimes called a "multi-cloud strategy," has emerged as a response to the growing recognition that even major cloud providers experience outages. By distributing workloads and backups across multiple providers, organizations can significantly reduce their vulnerability to provider-specific failures.
Cross-cloud architectures also address commercial concerns about vendor lock-in. When an organization's data and applications are confined to a single cloud ecosystem, the switching costs become prohibitively high, potentially leading to unfavorable pricing during contract renewals. By maintaining operational capability across multiple clouds, companies retain negotiating leverage while  enhancing their resilience posture.

The implementation of a cross-cloud 3-2-1 strategy typically follows a hierarchical approach, with data flowing from the primary environment to secondary and tertiary storage locations. A common pattern looks like this:

**TODO: Add Diagram (Ask David What that Should look like)**

Primary Cloud (AWS) → Secondary Cloud (Azure) → Cold Storage (GCP)

This architecture creates multiple layers of protection, each with its own recovery characteristics and cost profile. Let's examine the components of this approach in detail:

**Components:**

1. **Primary Environment (AWS)**

- Production workloads running in AWS
- Native Elastic Block Store snapshots and AMI backups (Copy 1)
- **RPO:** 1 hour

The primary environment hosts the production workloads and typically leverages the cloud provider's native backup capabilities. These tools are designed to create application-consistent snapshots that capture the state of virtual machines, databases, and storage volumes at specific points in time. The frequency of these snapshots determines the Recovery Point Objective, which in this example is set to one hour. Which means that in a disaster scenario, up to one hour of data might be lost.

2. **Secondary Cloud (Azure)**

- Cloud-to-cloud backup service
- Application data replicated to Azure Blob Storage (Copy 2, Media 1)
- Ready-to-deploy VM templates
- **RPO:** 4 hours, **RTO:** 1 hour for critical services

The secondary cloud environment serves as both a backup target and a potential recovery site. Rather than simply storing backup data, this tier often includes pre-configured resources that can be rapidly activated in the event of a primary cloud failure.
This might include virtual network configurations, security groups, and machine templates that mirror the production environment. The slightly longer RPO (4 hours) represents a trade-off between cost and data currency, while the aggressive RTO (1 hour) reflects the ready-to-activate nature of this environment.

3. **Cold Storage (GCP)**

- Archive-tier storage for long-term retention
- Monthly consolidated backups (Copy 3, Media 2)
- Write-once-read-many (WORM) protection
- **RPO:** 24 hours, **RTO:** 12+ hours

The third environment typically focuses on long-term retention and protection against widespread failures or security incidents. Something of a worst case scenario backup. By using a different cloud provider's archive storage, organizations further diversify their technology stack and eliminate shared dependencies.
The WORM (Write-Once-Read-Many) protection makes these backups immutable, providing an additional safeguard against ransomware or malicious insiders. The extended RPO and RTO values reflect the focus on disaster recovery rather than business continuity at this tier.

**Cross-Cloud Replication Challenges:**

- **Data Transfer Costs**

  - Inter-cloud data transfer can be expensive
  - **Solution:** Compression and deduplication to reduce transfer volume

- **Consistency**

  - Maintaining application consistency across platforms
  - **Solution:** Application-aware backup agents or database-native replication

- **Automation**

  - Complex orchestration of multi-cloud resources
  - **Solution:** Infrastructure as Code (IaC) templates for recovery

### RTO/RPO Optimization Techniques

Different architectures enable different recovery capabilities:

| **Architecture**           | **Typical RPO** | **Typical RTO** | **Cost** |
| -------------------------- | --------------- | --------------- | -------- |
| Backup Only                | 24 hours        | 4-24 hours      | $        |
| Backup + Replication       | 15 minutes      | 1-4 hours       | $$       |
| Continuous Data Protection | Near-zero       | 15-60 minutes   | $$$      |
| Synchronous Replication    | Zero            | Minutes         | $$$$     |

### Tiered Recovery Architecture

Most organizations implement a tiered approach based on workload criticality:

1. **Tier 0 (Mission Critical)**

- Always-on applications that cannot tolerate downtime
- Active-active deployment across multiple sites
- Synchronous replication
- **RPO:** 0, **RTO:** Minutes

2. **Tier 1 (Business Critical)**

- Important applications with moderate downtime tolerance
- Asynchronous replication with continuous data protection
- **RPO:** Minutes, **RTO:** < 4 hours

3. **Tier 2 (Important)**

- Applications that can tolerate longer recovery
- Daily backups with periodic replication
- **RPO:** 24 hours, **RTO:** 24 hours

4. **Tier 3 (Non-critical)**

- Weekly backups, minimal replication
- **RPO:** 1 week, **RTO:** Days
