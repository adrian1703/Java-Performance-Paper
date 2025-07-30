# Disaster Recovery

_Disaster Recovery (DR)_ encompasses the processes and procedures that ensure an organization's ability to recover from incidents that disrupt critical IT services or infrastructure, such as natural disasters, cyberattacks, or major hardware failures. While **High Availability (HA)** aims to minimize downtime within a region or data center by implementing redundancy, DR specifically addresses recovery from larger-scale events, potentially impacting physical locations or cloud regions.

---

## Key Components of a Disaster Recovery Plan

1. **Risk Assessment & Business Impact Analysis (BIA)**
    - Identify potential threats (e.g., natural disasters, cyberattacks, hardware failure, cloud region outages).
    - Evaluate the impact of each threat on business operations and prioritize applications/services by criticality.

2. **Recovery Objectives**
    - **RTO (Recovery Time Objective):** Maximum acceptable downtime before service must be restored.
    - **RPO (Recovery Point Objective):** Maximum acceptable data loss measured in time.
      - seconds vs hours is a big difference
    - Define objectives per application and database component.

3. **Backup Strategy**
    - Define backup frequency (e.g., daily full with hourly incremental).
    - Choose backup types: full, incremental, differential; clarify physical vs. logical backup for databases.
    - Store backups offsite or in the cloud; use immutable, encrypted storage where possible.
    - Automate backup jobs and verify backup integrity by frequent test restores.

4. **Disaster Recovery Sites**
    - **Cold Site:** Infrastructure available, but not provisioned or actively running.
    - **Warm Site:** Partially provisioned environment, some services pre-installed but not processing traffic.
    - **Hot Site:** Fully operational standby environment, ready for immediate failover with near-real-time data replication.
    - **Cloud / Multi-Region:** Leverage managed services or multi-region deployments for rapid scalability.

5. **DR Tools & Technologies**
    - Use virtualization, containerization, Infrastructure as Code (e.g., Terraform, **Ansible**, Jenkins) for reproducible DR environments.
    - Employ DR solutions from cloud providers (AWS, Azure, Google Cloud) for managed failover and backup orchestration.
    - **Automate** backup, failover, and restore processes **as much as possible**.

6. **Communication Plan**
    - Establish contact lists and notification procedures for incident response teams, executives, and external partners.
    - Define internal and external (e.g., customer) communication protocols and update channels during a disaster.

7. **Testing & Training**
    - Conduct regular DR drills (tabletop and simulation/failover tests).
    - Document results and update processes based on lessons learned.
    - Train staff on DR procedures and clarify roles and responsibilities.

8. **Documentation**
    - Maintain a clear, updated DR plan, with step-by-step recovery runbooks.
    - Store DR documentation where it is accessible both onsite and offsite.
    - Version control DR documentation, scripts, and infrastructure-as-code templates.

---

## Application Components Affected by DR

1. **Stateless Server (e.g., Tomcat)**
    - Easily replicated; redeploy using automated scripts or container images.
    - Critical dependencies: application configuration, environment variables, network security.

2. **Stateful (internal) Database (Postgres)**
    - Requires more complex recovery/replication strategies due to data consistency.

---

## Techniques & Implementation Patterns

1. **Internal Rerouting & Replication**
    - Clients/services automatically reroute to healthy servers.
    - Replication for both servers and databases; e.g., PostgreSQL streaming replication.
    - Use of service discovery and load balancers.

2. **Heartbeat & Health Checks**
    - Orchestration systems perform regular health checks.
    - On failure, automated recovery (restart/replace/redirect).

3. **Containers with Orchestration for Automatic Failover**
    - Container orchestration platforms (Kubernetes, Docker Swarm) automate failover, monitoring, and scaling.
    - StatefulSets (Kubernetes) can handle rolling updates and controlled failovers for stateful services like PostgreSQL.

4. **Database as a Cluster**
    - Use distributed relational DBMS (e.g., Citus, Postgres-XL) or replication strategies.
    - Primary/replica setups (leader/follower), or multi-leader topologies.
    - Combine storage-level and logical replication for flexibility.
    - Automate fencing and failback to avoid split-brain scenarios.
    - **Point-in-Time Recovery (PITR):** Configure PostgreSQL for PITR using WAL archive, ensuring time-accurate restoration.

5. **Infrastructure as Code**
    - Maintain versioned definitions of DR infrastructure for rapid rebuilding.
    - Tools: Terraform, Ansible, Pulumi.

6. **Monitoring & Alerting**
    - Implement DR-specific monitoring:
        - Backup success/latency/integrity.
        - Replication lag / health.
        - DRaaS tool states, application uptime.
    - Integrate alerts with incident response platforms (PagerDuty, OpsGenie).
    - Regular test restores and simulated failovers.

7. **Security Aspects**
    - Encrypt all backup data both in transit and at rest.
    - Protect DR resources and plans with role-based access control (RBAC).
    - Audit all DR activities and access events.

8. **Manually induced failures**
   - Seems odd but is often used in orchestration to randomly kill containers etc.
   - These tests failover and robustness of the system. 

---

## Distributed Databases: Key Concepts

- **Sharding/Partitioning:** Distributes data for parallelism and scale; beware of cross-shard consistency challenges.
- **Replication:** For failover and scaling reads; typically synchronous (for consistency) or asynchronous (for lower latency).
- **Leader/Follower:** Single writable node, replicas for reads/high availability; failover on leader loss managed by quorum.
- **Multi-Leader/Follower:** Multiple writable nodes, complex conflict resolution; useful for multi-region/multi-cloud.
- **Failover Process:** Quorum-based leader election 
  - etcd (distributed, reliable key-value store)
  - RAFT (log replication scheme) 
  - patroni (open-source cluster management tool for PostgreSQL)
- **Split-Brain Avoidance:** Implement fencing and strict election to prevent data divergence - should be done via cluster
  management tool.
- **Node Tolerance:** For n nodes, majority quorum needed (floor(n/2)+1).

---

## Example DR Procedures (Runbook Format)

### Example: Total Loss of Primary Database Node

1. Detect failure via monitoring (node unhealthy/replication broken).
2. Trigger failover to a replica (automated or manual, using repmgr/patroni/K8s operators).
3. Update application config/service discovery to point at new primary.
4. Verify application health and data consistency.
5. Initiate recovery of lost node and rejoin it as a new replica.

### Example: Restore from Backup (Point-in-Time)

1. Provision new PostgreSQL instance (automated via IaC).
2. Restore latest full backup.
3. Apply archived WALs up to the desired recovery point.
4. Validate integrity and start database instance.
5. Update routing verify client connectivity.

---

## Measuring & Improving DR

- Regularly test the DR plan and document results.
- Continuously improve based on test outcomes, technology advancements, and business growth.
- Track real RTO/RPO metrics.

## Personal opinion

1. Automate your complete deployment process - infrastructure as code 
   - deploying a new database to cluster or a new Tomcat Server should be as easy as 
     pushing a button or change to the configuration repository
2. switch to a database cluster management tool still using postgres as underlying db
   - partitioning is not necessary
   - use 3 nodes: Leader, Follower, Follower where Followers are full replicas using replicated log
3. Test yourself and your DR mechanisms by regulary terminating connection to a server/container