# Topology 1: Single Host (Basic)

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
  
