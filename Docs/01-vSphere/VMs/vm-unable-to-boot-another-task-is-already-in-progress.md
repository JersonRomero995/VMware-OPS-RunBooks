# 1 Problem Description

A production VM becomes unresponsive or has severe performance degradation. This negatively affects visibility of the infrastructure.

# 2 Observed Symptoms
Symptom	Possible Indicator
VM freezes, UI not responsive	VM console black, remote login fails
Tools disconnected	Monitoring agent shows inactive
High CPU/Memory usage	Slow application response
Disk/latency spikes	Slow I/O, services timeout
High ESXi host contention	Multiple VMs affected

# 3 Common Root Causes
Domain	Possible Cause
CPU	High Co-Stop, Ready %, CPU limit applied, contention
Memory	Ballooning, swapping, host overcommitment
Storage	High latency, datastore congestion, queue depth saturation
Network	Packet loss, incorrect MTU, bandwidth saturation
VM Config	Incorrect vCPU sizing, snapshot growth, thin provisioning impact
Tools/Drivers	VMware Tools outdated, paravirtual drivers missing
Application/OS	Hung processes, leaking application memory

# 4 Troubleshooting Workflow
🔍 Step A — Validate VM State
GUI
Check in vCenter > VM > Summary
Verify:
VM Tools Status
CPU/Memory usage
Network I/O
If console is responsive

CLI (from ESXi):
vim-cmd vmsvc/getallvms | grep -i <vm_name>
vim-cmd vmsvc/power.getstate <VMID>

🧠 Step B — Check ESXi Host Health

GUI:
vCenter > Host > Monitor > Performance
Metrics to check:
CPU Ready (%RDY > 10% = bad)
Co-Stop (>3% = vCPU sizing incorrect)
Memory ballooning or swapping
Disk latency (above 20-30 ms is bad)
CLI Commands:
CPU Ready:

esxcli vm process list
Check host CPU and memory:

esxtop
In esxtop:
Press c for CPU → Check %RDY, %CSTP
Press m for Memory → Check SWCUR, MCTLSZ
Press d for Disk → Check latency (DAVG/cmd, KAVG/cmd, %USD)
Press n for Network

Red Flags
%RDY > 10%
SWCUR > 0 (active swapping)
DAVG > 50ms

💾 Step C — Check Storage & Snapshots

List snapshots:
vim-cmd vmsvc/snapshot.get <VMID>
Check snapshot size:

ls -lh /vmfs/volumes/<datastore>/<vm_name>/
If snapshot is large (GB-level) → consolidate.

Virtual Machine Becomes Unresponsive with "Another task is already in progress" Error Due to Oversized Snapshot
https://knowledge.broadcom.com/external/article?articleNumber=399570

Powering off an unresponsive virtual machine on an ESXi host
https://knowledge.broadcom.com/external/article?legacyId=1004340

Unable to power off virtual machine on an ESXi host.
A virtual machine is unresponsive and cannot be forced off or stopped.
The host cannot access the virtual machine or unlock files.
After shutting down a virtual machine, vCenter Server shows the virtual machine as running.
There is no indication that a virtual machine is shut down.
Edit Settings option is unavailable for the virtual machine.
Unable to perform any task on vm
Attempting actions on the VMs outputs one or more of these errors:
A general system error occurred: Syscall kill returned error (-1) during vm termination attempt: ## with cartel id: ## and error message: No such process

Soap error 999. The operation is not allowed in current state

The attempted operation cannot be performed in the current state (Powered Off)

The request refers to an object that no longer exists or has never existed

Commands to run related to issues with snapshots
vim-cmd vmsvc/getallvms
vim-cmd vmsvc/power.getstate VMID
vim-cmd vimsvc/task_info haTask-2-vim.VirtualMachine.createSnapshot-1234567 --> to see task info
vim-cmd vimsvc/task_cancel task_id
vim-cmd vmsvc/power.shutdown VMID
vim-cmd vmsvc/power.off VMID

Best practices for using VMware snapshots in the vSphere environment
https://knowledge.broadcom.com/external/article/318825/best-practices-for-using-vmware-snapshot.html 


🌐 Step D — Check Network
From ESXi:
esxtop   # press n
Check:
Dropped packets (DroppedTx, DroppedRx)
High utilization
Ping test from ESXi:
vmkping <gateway>
vmkping <VM_IP>

⚙️ Step E — Check VM-Level OS Health
If console works:
Check CPU/memory usage in OS
Restart stuck monitoring service
Validate disk & network utilization
Look for application heavy load
Example Linux:

top
iostat -x
vmstat 5
df -h


Windows:

Task Manager
Resource Monitor
Event Viewer (Errors 1000, disk warnings)