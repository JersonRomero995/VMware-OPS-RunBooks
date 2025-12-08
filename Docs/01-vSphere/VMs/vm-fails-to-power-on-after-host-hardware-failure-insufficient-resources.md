# RUNBOOK – VM Fails to Power On After Host Hardware Failure (“Insufficient Resources”)
## 1. Overview

A host suffered a hardware memory failure. A Domain Controller VM was registered on that host.
When the host was removed from the cluster, the VM entry was removed as well. After re-importing the VM (via “Register VM”), powering it on fails with:
```
“Insufficient resources”
Sometimes accompanied by:
“Failed to lock the file”
“Could not power on because the swapfile cannot be created”
“Unable to allocate memory”
```
This runbook helps diagnose whether the issue is due to memory overcommit, corrupted VMX, reservation conflicts, or swapfile path issues.

## 2. Initial Validation
## 2.1 Validate Cluster and Host State

Ensure the target host is healthy, in connected state, and not reporting:

Memory degradation

PSOD logs

DIMM failures

NUMA node offline

Check:
```
esxcli hardware memory get
```
## 2.2 Validate Available Memory on Target Host

Power-on requires:

Free physical memory + ability to create the VM swap file.

SSH the host → run:
```
esxtop
```

Press m (memory):

Check:

Free Mem (MB)

Mem overcommit ratio

SWAP values

PSHARE, BALLON, SWAPUSED

Host must have at least VM configured memory available.

## 3. Validate VM Files After Re-import
## 3.1 Check VM Directory

SSH to the datastore or use datastore browser:

Look for:

VMX file

VMDK descriptor + flat file

Swap file not present yet (normal)

No orphaned .lck files

Check locks:
```
ls -lh /vmfs/volumes/<datastore>/<vmname> | grep -i lck
```

If locks exist → host may still believe VM is running on failed host.

Remove stale locks:
```
rm -rf *.lck
```

(Only if VM is confirmed powered off and files not in use.)

## 4. Check VMX for Reservation & Swapfile Errors
## 4.1 Retrieve VMX content
```
cat /vmfs/volumes/<datastore>/<vmname>/<vmname>.vmx
```

Look for:

## 4.1.1 Memory Reservation
sched.mem.max

memSize = "XXXX"

mem.MemZipEnable = "FALSE"

sched.mem.pin = "TRUE"


If reservation equals full memory, host must have 100% memory free.

Fix: Remove full reservation temporarily.

## 4.2 Swapfile Location Errors

If the VM or cluster used:
```
sched.swap.dir = "/vmfs/volumes/<wrong-path>"
```

or the cluster is set to Store swapfile in datastore different from host scratch.

If that datastore is unavailable → insufficient resources is triggered.

Fix:

Edit VM settings → Swapfile placement = With the VM.

Or update VMX manually:
```
vi /vmfs/volumes/<datastore>/<vmname>/<vmname>.vmx
sched.swap.dir = ""
```

Reload VMX:
```
vim-cmd solo/registervm /vmfs/volumes/<datastore>/<vmname>/<vmname>.vmx
```
# 5. Cluster-Level Troubleshooting
## 5.1 Admission Control

If they powered on the VM inside the cluster, HA Admission Control may block it.

Symptoms:

“Insufficient resources”

VM allowed on standalone host but not in cluster

Fix:

Temporarily disable Admission Control

Try power-on again

## 5.2 DRS Anti-Affinity Rules

Check for:

Must run on specific host

Cannot run on same host as other VMs

Fix: Disable rules temporarily.

## 6. Host-Level Troubleshooting (Critical)
## 6.1 Verify Host is Not in MM or Entering Maintenance
vim-cmd hostsvc/hostsummary | grep -i maintenancemode

## 6.2 Verify Host Can Create Swap

On the ESXi host:
```
vmkfstools -P /vmfs/volumes/<datastore>
```

Look for:

Free space

Access OK

No APD/PDL

If datastore for swap is not reachable → VM will not power on.

## 6.3 Memory Compression or Ballooning Exhausted

If ballooning or compression is already high → power-on may fail.

Check:
```
esxtop → m
```

If MEMCTL > 0 and SWAP > 0 you're out of memory.

## 7. Repair Procedures
## 7.1 Clear Stale Processes

Sometimes failed host crash leaves ghost entries.

Check running VMs:
```
vim-cmd vmsvc/getallvms
```

Check for ghost process:
```
ps -c | grep -i vmx
```

Kill if necessary:
```
kill -9 <pid>
```
## 7.2 Re-register VM Cleanly

Often the VMX gets corrupted during host removal.

Remove VM from inventory

Verify no .lck files

Re-register:
```
vim-cmd solo/registervm /vmfs/volumes/<datastore>/<vm-folder>/<vm-name>.vmx
```
## 7.3 Fix Memory Reservation

If the VM was a Domain Controller → sometimes it's created with full reservation.

Edit settings in vCenter:

Edit VM

VM Options → Advanced → Memory

Remove or reduce reservation

## 7.4 Try Powering On With CPU Compatibility Disabled

If coming from a failed host with different CPU features:
```
vim-cmd vmsvc/power.on <vmid>
```
## 7.5 Create a New VMX (Last Resort)

If VMX is corrupted beyond repair:

Create a new dummy VM

Remove its VMDK

Attach the DC’s existing VMDK

Apply same OS settings

Power on

This preserves the disks and data safely.

## 8. Key Logs to Review

On ESXi host:

VMkernel
```
tail -f /var/log/vmkernel.log
```

Look for:

Admission Control failures

Swap creation errors

Memory allocation failures

Lock conflicts

Hostd
```
tail -f /var/log/hostd.log
```
vpxa
```
tail -f /var/log/vpxa.log
```
If original host had memory errors

Retrieve logs from failed host (if possible):
```
/var/log/vmkernel.log*

/var/log/syslog.log
```
## 9. Root Cause Patterns for This Scenario

✔ Memory reservation exists but host cannot satisfy it

Very common with Domain Controllers.

✔ Swapfile location was pointing to datastore affected by the failed host

Seen when HA/DRS rules use "Store swap files in cluster datastore".

✔ Residual locks from the failed host due to sudden removal

✔ VMX corruption after host removal from cluster

✔ Host actually does NOT have enough physical memory

Even if it looks free, NUMA restrictions may apply.

## 10. Recommended Recovery Sequence

Use this order to guarantee fastest recovery:

Check host memory free space (esxtop)

Remove stale lock files

Validate VMX → remove memory reservation

Ensure swapfile location is valid

Re-register VM

Power on directly on host (not in cluster)

If still fails → create new VM and attach existing VMDKs
