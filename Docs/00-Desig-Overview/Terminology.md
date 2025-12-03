# Terminology

## Purpose
This document defines key terms and concepts used in VMware operations runbooks to ensure consistent understanding and execution of procedures.

## VMware Infrastructure Terms

**vCenter Server**: Centralized management platform for VMware vSphere environments

**ESXi Host**: Bare-metal hypervisor that runs virtual machines

**vSphere**: VMware's virtualization platform suite

**Datastore**: Storage repository for virtual machine files, ISOs, and templates

**Resource Pool**: Logical abstraction for managing CPU and memory resources

**vMotion**: Live migration of running VMs between ESXi hosts

**DRS (Distributed Resource Scheduler)**: Automated load balancing across cluster hosts

**HA (High Availability)**: Automated VM restart on alternate hosts during failures

**Snapshot**: Point-in-time state of a virtual machine

**Template**: Master copy of a VM used for cloning

**Cluster**: Group of ESXi hosts sharing resources and providing HA/DRS

## Operations Terms

**Runbook**: Step-by-step documented procedure for operational tasks

**Playbook**: Collection of related runbooks for a specific scenario

**Incident**: Unplanned interruption or degradation of service

**Change Window**: Scheduled maintenance period for infrastructure changes

**Rollback**: Process to revert changes to previous known-good state

**Health Check**: Routine verification of system status and performance

**Escalation**: Process of forwarding issues to higher-level support

## Monitoring & Alerting

**Threshold**: Predefined limit that triggers an alert when exceeded

**Baseline**: Normal operating metrics used for comparison

**Alert**: Notification of a condition requiring attention

**Event**: Logged occurrence in the infrastructure

**Performance Metrics**: Measurable values indicating system health (CPU, memory, disk I/O)

## Backup & Recovery

**RPO (Recovery Point Objective)**: Maximum acceptable data loss measured in time

**RTO (Recovery Time Objective)**: Maximum acceptable downtime

**Full Backup**: Complete copy of all data

**Incremental Backup**: Backup of data changed since last backup

**Replication**: Continuous copying of VMs to secondary site

## Network Terms

**vSwitch (Virtual Switch)**: Software-based network switch within ESXi

**Port Group**: Logical grouping of ports with common network policies

**VLAN**: Virtual LAN for network segmentation

**vNIC**: Virtual network interface card

**Distributed Switch**: vSwitch spanning multiple ESXi hosts

## Storage Terms

**VMFS (Virtual Machine File System)**: VMware's clustered file system

**NFS**: Network File System for shared storage

**SAN (Storage Area Network)**: Dedicated high-speed network for storage

**LUN (Logical Unit Number)**: Block of storage presented to hosts

**Thin Provisioning**: Dynamic allocation of storage space

**Thick Provisioning**: Pre-allocated storage space

## Automation Terms

**PowerCLI**: PowerShell cmdlets for VMware automation

**vRealize Automation**: VMware's cloud automation platform

**Ansible**: Configuration management and automation tool

**Orchestration**: Automated coordination of multiple tasks

## Security Terms

**vMotion Encryption**: Encrypted live migration of VMs

**VM Encryption**: Encryption of VM files at rest

**Role-Based Access Control (RBAC)**: Permission assignment based on user roles

**SSO (Single Sign-On)**: Centralized authentication mechanism

## Acronyms

- **VM**: Virtual Machine
- **HA**: High Availability
- **DRS**: Distributed Resource Scheduler
- **SRM**: Site Recovery Manager
- **NSX**: VMware's network virtualization platform
- **vSAN**: VMware Virtual SAN
- **SDDC**: Software-Defined Data Center
- **API**: Application Programming Interface
- **CLI**: Command Line Interface
- **GUI**: Graphical User Interface

## Runbook-Specific Terms

**Prerequisites**: Required conditions before executing a runbook

**Post-Execution Validation**: Steps to verify successful completion

**Decision Point**: Step requiring human judgment or approval

**Automated Step**: Action performed by scripts without manual intervention

**Manual Step**: Action requiring human execution

**Rollback Procedure**: Steps to undo changes if issues occur
