# Intro
VCF 9.0 gives new features that support building, operating, consuming and protecting moderm private cloud infra

Building a moderm private cloud 

## Shipping with VCF 5.2
'''
-Quick start App for VCF deployment
-MIxed lifeclycle management
-Offline depot
'''

## VMware Cloud Foundation 9.0
'''
-VMware cloud foundation installer 
-VCF data path enhancements
-Cost effective NVMe memory tiering
-VMware cloud Foundation import
-Performant fleet management capabilities
-Easy tenant deployment and management
'''

## Operating a moderm private cloud 
### VCF 5.2
'''
-vSAN HCI strech cluster and storage Clusters
-Easy adoption of virtual networks
-VCF EntraID support
'''

### VCF 9.0
'''
-VCF operations console
-Enhanced VCF diagnostics with skyline
-Improve Storage Efficiency with vSAN goblal dedup
-Centralized license management
-Identity and cerificate management 
'''

## Consumption-ready experience 
### VCF 5.2
'''
-IaaS and IaaC services
-Kubernetes support embedded in vSphere
-Independent container support
'''

### VCF 9
'''
-One consumption experience
-Virtual private clouds (VCPs)
-Native multu-tenancy
-Easy IaaC and container management
-Self-service catalog and services
-Enhanced private chargeback and Showback
'''

## Secure and resilient Private Cloud
### VCF 5.2
'''
-Critical in-Service ESX patching
-Flexible, asynchronous upgrades
-Extended data protection features
'''

### VCF 9.0
'''
-Security operations dashboard
-Platfrom hardening and security
-Unified data and ransomware protection
-Configuration compliance monitoring
-Application troubleshooting and insights 
'''

## What's new in VCF architecture 
'''
A VCF private cloud is the highest level of management and consumption for underlying software-define data center (SDDC) resources.

A VCF private cloud contains one or more VCF fleets:
-A VCF fleet is an enviroment managed by a single set of fleet-level management components.
-A VCF fleet contains the following components:
    - One VCF operations instance
    - One VCF Automation instance
    - One or more VCF instances

- A VCF instance delivers the SDDC resources for a VCF fleet, which contains compute, storage, and network resources.

- A VCF instance comprises a management domain and, optionally, one or more workload domains.
'''

## What's new in VCF Installation
'''
Benefits of VCF installer include:
-Simplifies the deployment of VCF or vSphere Foundations for new and existing deployments
-Eliminates the need for individual VCF component installations
-Reduce risks by using a validated topology 
'''

## What's new in VCF licensing
'''
VCF 5.2                                 VCF 9.0
License key                             License File
License per component                   Single license for all components 
Does nogt require usage data            Require usage data
No central visibility/control           Central sonsumption visibility and control
Manual reactivation on renewal          Automated reactivation on renewal
Major version specific                  No more license upgrades or downgrades
Managed by broadcom support portal      Managed by VCF bussines services console
60 day evaluation period                90 day evaluation period
Per host Licensing                      Per vCenter licensing
'''

## New license process

'''
1- Register VCF operations
2- Select products and quantity 
3- Download the license file
4- Assign to vCenter
'''

## New in VCF compute
'''
-Memory optimization with NVMe
    -Helps to optimize server resource usage
    -Addresses core to memory imbalance
    -Provides more in memory computing
    -Runs more worloads with better CPU utilization

-Increased uptime for AI/ML workloads
    -Performance improvements from 10 Gbps to 60 Gbps(6x)
    -Less than 1 second stun timer for an 8 GB vGPU profile

-Mixed vendor cluster
    -Assign additional images manually or automatically
    -Have multiple hardware support managers per cluster image
    -Create up to four additional images per cluster image
    -Support multiple server models in a single cluster image

-Virtualized hardware innovations
    -Up to 960 vCPU
    -latest CPU support
    -Latest guest OS support
    -AMD SEV-SNP and intel TDX
    -TPM spec 2.0 rev 1.59
    -New guest customization APIs
    -Virtual NVMe namespaces write protection

-Decoupling of supervisor updates from vCenter
    -Update the supervisor control plane independently of vCenter
    -Subscribe to BC depot or manager an offline contect library 
    -Get faster delivery of supervisor releases

-Live patching enhancements 
    -Live patch is extended to VMkernel, user space and NSX components
    -ESX service restarts are required but they are not disruptive
    -fast-supsend-resume remediates VMs
    -No evacuations is required
'''

## New in VCF storage
'''
-Network traffic separations for vSAN storage clusters
    -vSAN cluster traffic is isolated from vSphere clusters that access the data store on the vSAN storage cluster
    -Dedicated VMKernel interfaces are provided for: vSAN cluster traffic and vSphere client clusters mouting vSAN storage cluster data store
    -Network traffic to and from client cluster is minimized

-Enhanced scalability with vSAN file services
    -Shares supported per cluster in vSAN file services are twice the number of the previous release
    -Cluster limit can be a combination of NFS and SBM shares

-Enhanced flexibility, uptime, and operational task for stretched clusters
    -VCF 9.0 provides support for vSAN storage clusters and new site maintenance and site takeover capabilities
'''

## New in VCF networking
'''
-Clear role definition that allows self0service networking in virtual private clouds
    -Enterprise admin: owns the physical infra
    -Project admin: A tenant with no direct access to the physical network
    -VCP admin: end user with access to self-service networking

-vCenter UI workflow for deploying NSX edges
    -The user is guided through entering the require parameters
    -Graphical representation is generated in real time

-High performance switching
    -Does not require tuning
    -Provides a 50-70% increase performance (for both bandwith and packets per second)
    -Includes EDP driver with improved pNIC queue distribution
    -EDP dedicated mode can be configured, as well as the original standard VDS mode
'''

## VCF operations console
'''
Simplifies cloud operations 
    -Quick deployment
    -Proactive security and insights
    -Simplified management and life cycle
'''
### VCF automation
'''
-Moderm cloud interface
-Tenat management
-governance and policies
'''












