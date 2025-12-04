# Runbook: ESXi Host Latency Only Inside a Specific Cluster (vSphere 8)

## Scenario:
One ESXi host experiences high latency only when it is inside a specific cluster. When the same host is moved (Disconnected → Removed → Added) to another cluster in the same vCenter, the latency disappears.

# 1 Validate ESXi Host Health

SSH → run:
```
esxcli hardware pci list
esxcli hardware cpuid get
esxcli hardware ipmi sel get
esxtop (press d for disk, n for network)
```
Check:

CPU ready %

Storage latency (DAVG/KAVG/GAVG)

Packet drops in network

If the host only misbehaves inside the cluster, it strongly indicates cluster-level configuration, not host hardware.

# 2 Troubleshooting Deep Dive
# ✅ 2.1 vSAN Enabled Cluster (Most Common Root Cause)

If the problematic cluster has vSAN enabled:

# 2.1.1 Check vSAN Health

vCenter → Cluster → vSAN → Monitor → Health & Performance

Check for:

Congestion

Resync throttling

Disk group unhealthy

vSAN network MTU mismatch

Incorrect vSAN NIC teaming

# 2.1.2 Check ESXCLI vSAN Stats

SSH → run:
```
esxcli vsan network list
esxcli vsan perf stats get
esxcli vsan health cluster list
```
# 2.1.3 Compare with “Working” Cluster

If Cluster B does not run vSAN, the behavior difference makes sense.

Possible Fixes

Fix MTU mismatch (9000 vs 1500)

Verify vSAN vmkernel traffic

Correct vSAN teaming (should avoid LACP in many setups)

Validate multicast/unicast config depending on vSAN version

# ✅ 2.2 DRS or CPU Scheduling Acting Differently

If DRS is configured aggressively or there is contention:

2.2.1 Check CPU Scheduler

SSH → run:

esxtop


Check:

%RDY → should stay under 5%

%CSTP (co-stop for large vCPU VMs)

%MLMTD indicates CPU limits

# 2.2.2 Cluster-A Settings to Compare

In vCenter:
Cluster → Configure → vSphere DRS

Check:

Automation level

Scalable shares enabled/disabled

Any CPU or memory reservations applied at resource pool level

Any limits configured

Possible Fixes

Disable/enable DRS

Fix resource pool limits (common silent killer)

Adjust VM reservations

# ✅ 2.3 Resource Pools Misconfiguration (one of the top causes)

Resource pools with incorrect limits can create massive latency for one host while other clusters don’t have these pools.

Validation

Go to Cluster → Resource Allocation → Resource Pools → Check:

CPU Limit

Memory Limit

Expandable Reservation

A limit = throttling → causes latency.

Fix

Remove limits or recreate the resource pool tree.

# ✅ 2.4 EVC Mode Inconsistency

If cluster A has EVC enabled and cluster B does not, the host might run with CPU masking causing scheduling slowdowns.

Check

Cluster A → Configure → vSphere EVC
Compare with Cluster B.

Fix

Raise/lower EVC to match CPU generation

Disable EVC temporarily to test

Ensure hosts all share same CPU family compatibility

# ✅ 2.5 vSphere HA Heartbeat/Agent Issues

Faulty HA configuration can create cluster-level latency due to repeated agent retries.

Check HA Status

SSH:
```
/etc/init.d/vpxa status
/etc/init.d/hostd status
/etc/init.d/fdm status
```

vCenter:
Cluster → Monitor → vSphere HA

Fix

Reconfigure cluster for HA

Reinstall HA agent on affected host (Remove from cluster → Add again)

# 2.6 Storage Policy or Datastore Connectivity Bound to Cluster

Sometimes storage datastores are attached to a cluster via SPBM policies, multipathing, or specific network paths.

Check Storage Paths

SSH:
```
esxcli storage core device stats get
esxcli storage nmp device list
esxcli storage core path list
```

Compare:

PSP (Round Robin, MRU, Fixed)

ALUA states

Active paths

Fix

Match PSP with other clusters

Fix multipath policy

Validate iSCSI/NFS/vSAN vmkernel binding

# 3 Step-by-Step Troubleshooting Workflow
# Step 1 — Compare Cluster Configurations

Export cluster settings for A and B and diff:
Cluster → Configure → Quickstart → Review Hosts

Check for:

HA

DRS

EVC

vSAN

Admission Control

Resource Pools

# Step 2 — Put Host in Maintenance Mode in Cluster A

Check if latency persists without running VMs.

If latency persists ➜ cluster-level network/storage config.
If latency disappears ➜ VM/DRS/resource-pool issue.

# Step 3 — Check Networking
```
esxcli network nic list
esxcli network nic stats get
esxcli network vswitch standard list
```

Match MTU, VLAN, NIC teaming with cluster B.

# Step 4 — Check Storage Latency on Cluster A
esxtop → press d


Check DAVG, KAVG, GAVG.
Compare with cluster B.

# Step 5 — Review vCenter Agent Logs

On ESXi:
```
cat /var/log/vmkernel.log
cat /var/log/hostd.log
cat /var/log/vpxa.log
cat /var/log/fdm.log
```

Search for:

APD/PDL

SCSI aborts

NIC drops

vSAN congestion

# 🧪 Next Actions (Fast Diagnostic Path)
# 🔎 1. Test MTU (most important)
vmkping -I vmk0 -s 8972 x.x.x.x


If fail → MTU mismatch = primary cause.

# 🔎 2. Check NIC errors
esxcli network nic stats get -n vmnic0


If rcvdrops or txdrops > 0 → physical switch issue.

# 🔎 3. Compare vSwitch or vDS settings between clusters

Same MTU?

Same VLAN?

Same LACP settings?

Same load-balancing policy?

# 🔎 4. Trace a route to verify path difference
vmkping --netstack=default x.x.x.x

#🚀 What I Would Do If I Were Troubleshooting Live (Step-by-Step)
#Step 1: Compare MTU

Bad host:

esxcfg-vmknic -l


Good host:

esxcfg-vmknic -l


→ Confirm if Cluster A = 9000 && Cluster B = 1500.

# Step 2: Check NIC link state
ethtool vmnic0

# Step 3: Check VLAN trunking on DVS

Go to vCenter → Networking

Compare Cluster A dvSwitch vs Cluster B dvSwitch.

# Step 4: Ask network team

Which switch → which ports → which VLAN trunk → which MTU?

# Another possible situation from the network end and MTU mismatch if the vmkping result is similar to this:

Look at your test:

vmkping -I vmk0 -s 8972 <target host with issues>


Results:

66% packet loss

~170 ms latency

Only 1 packet succeeded

This only happens inside that one cluster

Good cluster = zero issues

This might confirm:

# ❌ The physical network links/uplinks/VLAN trunk for that cluster are NOT supporting jumbo frames (MTU 9000), even though the ESXi side expects MTU 9000.
#✔ The other cluster (working cluster) does have a clean jumbo frame path.
🎯 What This Means

Your cluster has at least one of these problems:

# 1. Physical switch ports for Cluster A are set to MTU 1500

While ESXi vmkernel interfaces (vmk0/vmk1/etc.) are configured for MTU 9000.

# 2. The VLAN used by that vmkernel path is not configured for jumbo frames

Even if other VLANs are.

# 3. LACP or port-channel on Cluster A switches is incorrectly configured

— mismatched physical ports
— one port with MTU 1500, the others 9000
This causes packet fragmentation + loss + latency spikes.

# 4. Cluster A uses a different DVS or different Uplink Profile (Netstack)

One profile has MTU 1500, the other MTU 9000.

# 5. A misconfigured router or firewall between hosts

If the gateway or intermediate device does not support MTU 9000.

# 🧪 How To Confirm Which Device Is Dropping the Packets
# Step 1 — Find the vmkernel you used
esxcfg-vmknic -l


Identify:

vmkX

VLAN ID

MTU

Compare with the working cluster.

# Step 2 — Check the switch uplinks used by that vmkernel

On the bad host:

esxcfg-vswitch -l


If using vDS:

esxcli network vswitch dvs vmware list


Look for:

Uplink assignment

MTU on vDS

VLAN trunking list

# Step 3 — Verify MTU on vmnic
esxcli network nic list


If speed/duplex is weird:

100/full

1000/half

flapping

→ Another strong indicator.

# Step 4 — Check NIC packet drops
esxcli network nic stats get -n vmnicX


If you see high txDrops or rxDrops → physical network mismatch.

🛠️ Fix (Network Team Required)

# If all path down error messages are found
This means the host is losing ALL its paths to a datastore, even temporarily, and that can absolutely cause:

high latency

VM freezes

host lockups

vSAN resync storms

vMotion failures

ICMP jitter (because kernel is stuck waiting for I/O)

So now we treat this as a storage path failure, not only network.

Here is exactly what you can perform.

# ✅ 1. Confirm APD in Logs

The APD message is always in vmkernel.log.

Search for:
```
grep -i "APD" /var/log/vmkernel.log
grep -i "All Paths Down" /var/log/vmkernel.log
grep -i "NMP" /var/log/vmkernel.log
```

Common APD patterns:
```
NMP: nmp_DeviceAttemptFailover: Retry world
APD Timeout
No Paths to device
WARNING: Path status changed to Dead
Device <naa.xxx> in APD state
```

This tells you:

which datastore/device

when the APD started

how long it lasted

which paths died

# ✅ 2. Identify the Affected Device

Run:

esxcli storage core device list


Find the device (naa.xxx) reported in the APD log.

Then check path state:

esxcli storage core path list -d <naa.xxx>


APD symptoms:

All paths = Dead

No active I/O

Path state changing repeatedly

Recoverable after failover

# ✅ 3. Determine the Type: APD vs PDL

Both are fatal but different:

APD (All Paths Down) = ESXi thinks storage is gone but doesn't know why.

Temporary → recoverable.

PDL (Permanent Device Loss) = Storage array TOLD ESXi “I’m gone.”

Not recoverable.

Check:

grep -i "PDL" /var/log/vmkernel.log

# ✅ 4. Actions You Can Perform Immediately
# ✔ 4.1 Restart I/O stack on ESXi

This often clears stale paths:
```
/etc/init.d/devfs restart
/etc/init.d/vmkernel-storage-migrated restart
/etc/init.d/storageRM restart
```

Do NOT reboot at this stage unless required.

# ✔ 4.2 Rescan Storage
```
esxcli storage core adapter rescan --all
esxcli storage core device rescan --all
```

This forces ESXi to rediscover paths.

# ✔ 4.3 Check Network Backing for iSCSI, NFS, or vSAN

Since you’ve shown packet loss and MTU issues, this is likely the root cause of APD.

Check vmkernel interfaces:

esxcfg-vmknic -l


Check NIC errors:

esxcli network nic stats get -n vmnicX


If ANY rx/tx errors increase → storage traffic is being dropped → leads to APD.

# ✔ 4.4 Check iSCSI Sessions (if using iSCSI)
esxcli iscsi session list


Symptoms:

Sessions flapping

Missing portals

Connection timeout

# ✔ 4.5 Check NFS Mount State (if using NFS)
esxcli storage nfs list


Symptoms:

Inactive state

Timeouts

# ✔ 4.6 For vSAN Clusters
esxcli vsan health cluster list


Look for:

vSAN network down

Disk group offline

Resync throttling due to MTU issues


