# Topology 1: Single Host (Basic)

## 1.1 Description
Simplest VMware setup with a single ESXi host managed by vCenter (optional). Suitable for
development, testing, or small labs.

## 1.2 Components
• 1× ESXi Host
• 1× vCenter Server Appliance (VCSA) – optional, can run as VM on the host
• Local or DAS (Direct Attached Storage)
• Standard vSwitch for networking

## 1.3 Architecture

```mermaid
flowchart TB
    subgraph "Single ESXi Host"
        direction TB
        ESXi[ESXi Host + 2 cores, 32 GB+ 8 RAM]
        
        subgraph "Virtual Machines"
            VM1[VM 1]
            VM2[VM 2]
            VM3[VM 3]
            vCSA[vCenter ServerAppliance - Optional]
        end
        
        subgraph "Storage"
            LocalStorage[(Local Storage or DAS)]
        end
        
        ESXi --> VM1
        ESXi --> VM2
        ESXi --> VM3
        ESXi --> vCSA
        ESXi --> LocalStorage
    end
    
    style ESXi fill:#4A90E2
    style vCSA fill:#F5A623
    style LocalStorage fill:#7ED321
```

## 1.4 Limitations
× No high availability
× Single point of failure
× Limited scalability
× No vMotion capability (requires shared storage)


