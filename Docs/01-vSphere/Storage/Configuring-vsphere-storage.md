# About Datastore 

A data store is a logical unit that can use space on one or more physical storage devices 
Datastores are used to hold data such as VMs, VM templates, and ISO images

vSphere supports the following types of datastores:
-VMFS
-NFS
-VSAN
-vSPhere virtual volumes 

# Datastore access methods
Block-backed storage:
-Store data as blocks (sequence of bytes)
-Used on local storage
-Used on storage Area Networks (SANs) and accessed through either iSCSI or Fibre channel
-Used by VMFS, VSAN, and vsphere virtual volumes datastores

File-backed storage:
-stores data hierarchically in files and folders
-Used on network-attached storage (NAS)
-Used by NFS and vSphere Virtual Volumes datastores

# Datastore contents 
File-base datastores:
-A VM consists of a set of files
-Each VM has its own directory
-VMFS and NFS datastores hold files

Object-based datastores:
-A VM consists of a set of data containers called objects
-VSAN and vSphere virtual volumes datastores hold objects

# Datastore summary

Datastore type            DataStore access method    Datastore contents
VMFS                      Block access               Files
NFS                       File access                Files
vSAN                      Block access               Objects
vSphere vitual volumes    Block or file access       Objects


Datastore type                  VMFS            NFS         vSAN            vSphere virtual volumes

Transport       Direct attached,fc,FCoE,iSCSI  Ethernet  Direct attached    FC/Ethernet

Backing         Disk            Lun Lun Lun    File System  vSAN cluster    Storage container

