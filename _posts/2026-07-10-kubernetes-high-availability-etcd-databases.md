---
title: "Kubernetes High Availability in Practice: etcd Quorum, Storage and Database Replication"
description: "High availability is a system property, not a node count. This guide examines control-plane quorum, storage recovery, database replication and operational validation."
categories: [Platform Engineering, Kubernetes]
tags: [kubernetes, rke2, etcd, high-availability, longhorn, cnpg, disaster-recovery]
---

A Kubernetes cluster can look healthy while containing several single points of failure. Multiple worker nodes do not make the control plane highly available. Replicated storage does not guarantee recoverability. Two database instances do not automatically provide a safe failover path.

High availability is a property of the complete service: control plane, scheduling, networking, storage, database, deployment system and the operational procedures used during failure.

## Control-plane availability begins with quorum

In an etcd-backed Kubernetes distribution such as RKE2, control-plane design must respect consensus. A three-member etcd cluster can lose one member and continue operating. A two-member cluster cannot safely tolerate the loss of either member because a majority is required.

This is why adding a third controller is not simply “one more server.” It changes the failure model.

The members should be placed across independent failure domains where possible. If three controllers share the same hypervisor, storage array and power source, they protect against a process or virtual-machine failure but not against the infrastructure failure that matters most.

Operational checks should confirm:

- all etcd members are healthy and participating;
- member latency is stable;
- time synchronization is correct;
- snapshots are current and stored outside the cluster;
- certificates and disk capacity are monitored;
- the team has rehearsed member replacement and snapshot restoration.

A snapshot that has never been restored is evidence of a backup process, not evidence of recoverability.

## Separate availability from capacity

Clusters are often sized for average utilization and then expected to survive a node loss. This is mathematically inconsistent. If workloads consume nearly all available CPU or memory, Kubernetes may keep the control plane alive after a failure but have nowhere to reschedule applications.

Capacity planning should reserve enough headroom for the declared failure scenario. Pod disruption budgets, topology spread constraints and anti-affinity rules are only effective when spare capacity exists.

Critical platform components also need deliberate placement. DNS, ingress controllers, metrics components and GitOps controllers should not all land on one worker simply because the scheduler found space there.

## Storage replication is not a backup

Distributed block storage systems such as Longhorn improve node-failure tolerance by maintaining replicas. They do not replace backups.

A logical deletion, corrupted write or application error can be replicated perfectly to every copy. A cluster-wide control-plane incident can also make healthy replicas difficult to attach until the infrastructure state is repaired.

Treat the following as separate layers:

- replicas for immediate infrastructure failure;
- snapshots for fast rollback within the storage system;
- backups exported to an independent location;
- application-aware backups for databases;
- documented recovery procedures that define order and ownership.

During a severe incident, recovery order matters. Restoring applications before the storage and control-plane state is stable can create additional writes and make diagnosis harder. Freeze unnecessary change, preserve evidence, validate volumes and recover the smallest critical service path first.

## Database high availability needs an explicit model

Running PostgreSQL in Kubernetes requires more than placing a database process in a StatefulSet. An operator such as CloudNativePG can manage streaming replication, failover, backups and lifecycle operations, but the team still owns the availability objectives.

For each database cluster, define:

- the number of instances and failure domains;
- synchronous or asynchronous replication expectations;
- recovery point and recovery time objectives;
- failover behavior and client reconnection;
- backup retention and restore validation;
- maintenance and upgrade procedures.

A primary plus one replica removes a single-instance failure, but it does not provide the same tolerance as a larger topology. The right design depends on business criticality, write latency and infrastructure cost.

Applications must also tolerate failover. Connection pools, DNS caching and transaction retry behavior can turn a successful database promotion into a prolonged application outage if they are not tested together.

## GitOps must survive the incident it is meant to repair

GitOps is often described as a recovery mechanism: the desired state is stored in Git and the controller reconciles the cluster. That statement assumes the controller itself is available.

Running ArgoCD in a highly available configuration protects the deployment control plane from a single pod or node failure. Repository credentials, cluster secrets and application definitions must also be backed up or reproducible through documented procedures.

Git remains the source of truth, but emergency recovery sometimes requires carefully controlled manual actions. The important discipline is to record those actions, reconcile them back into Git and remove temporary exceptions after stability returns.

## Observe leading indicators

Availability work should not begin only after a node disappears. Useful leading indicators include:

- etcd database size, proposal latency and leader changes;
- node disk pressure and filesystem growth;
- storage replica health and rebuild duration;
- database replication lag and backup age;
- unschedulable pods after simulated node loss;
- certificate expiry and time drift;
- GitOps reconciliation failures.

Alerts should describe an actionable condition. “Pod restarted” is often noise; “database has no healthy replica” requires immediate action. Severity should reflect user impact and remaining redundancy, not only the existence of an error.

## Run failure exercises

Documentation improves recovery, but controlled exercises reveal hidden coupling. Useful scenarios include:

1. drain and shut down one worker;
2. remove one controller from service;
3. fail over a database primary;
4. restore a database into an isolated namespace;
5. recover a volume from backup;
6. rebuild a platform component from Git;
7. verify that customer-facing monitoring detects the impact.

Record the actual recovery time, unexpected dependencies and manual decisions. Turn every finding into a tracked improvement rather than leaving it in an incident chat.

## The operational definition of HA

A platform is highly available only when it can lose a declared component, continue or recover within its objective, and explain its current state through reliable telemetry.

Three controllers, replicated volumes and database replicas are important building blocks. The real architecture is the combination of those components with capacity, backups, tested procedures and people who know which actions are safe during pressure. High availability is achieved when the failure model is explicit and repeatedly validated.
