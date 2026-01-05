# 🛠️ vSphere HA Troubleshooting Runbook
## Scope

This runbook applies to VMware vSphere High Availability (HA) issues affecting:

Cluster availability

VM restart failures

Host isolation

HA agent (FDM) failures

Datastore heartbeats

Network partition events

## 1️⃣ Initial Triage
### 1.1 Identify the Symptom

Check vCenter → Cluster → Monitor → vSphere HA → Issues

Common messages:

vSphere HA agent unreachable

Insufficient resources to satisfy configured failover level

Host is isolated

Cannot find suitable master

Datastore heartbeat lost

### 1.2 Validate Impact

Are VMs running but HA alerts present?

Did a host fail and VMs did not restart?

Are multiple hosts affected or just one?

## 2️⃣ Verify Cluster HA Configuration
### 2.1 HA Enabled

Cluster → Configure → Services → vSphere Availability

Ensure:

HA is ON

Admission Control matches design

### 2.2 Admission Control Check

Policy type:

Host failures cluster tolerates

Percentage of cluster resources

Confirm no resource exhaustion:

CPU / Memory reservations

VM-level reservations blocking restarts

## 3️⃣ HA Agent (FDM) Status Check
### 3.1 Host Level

On affected ESXi host:
```
/etc/init.d/vmware-fdm status
```

Restart if needed:
```
/etc/init.d/vmware-fdm restart
```
### 3.2 Reconfigure HA (Non-Disruptive)

From vCenter:

Right-click host → Reconfigure for vSphere HA

If many hosts:

Disable HA at cluster level

Re-enable HA (forces full agent redeploy)

⚠️ Do not do this during active failover unless required

## 4️⃣ Network & Isolation Validation
### 4.1 Management Network Health

Check:

vmk0 status

Correct VLAN

Physical NIC redundancy
```
esxcli network nic list
esxcli network ip interface list
```

### 4.2 Isolation Address

Cluster → HA → Advanced Settings

Verify:

das.isolationaddress1

Should be a reachable gateway or core IP

Ping from ESXi shell:
```
vmkping <isolation_ip>
```

If isolation IP is unreachable → false isolation events

## 5️⃣ Datastore Heartbeat Validation
### 5.1 Heartbeat Datastores

Cluster → Monitor → vSphere HA → Datastore Heartbeats

Minimum:

2 accessible shared datastores

### 5.2 Validate from Host
```
esxcli storage vmfs extent list
esxcli storage filesystem list
```

Look for:

APD / PDL

Datastore inaccessible or high latency

## 6️⃣ Master / Slave Election Issues
### 6.1 Identify HA Master

Cluster → Monitor → vSphere HA → Summary

If no master:

Usually caused by:

Management network partition

DNS resolution issues

MTU mismatch

### 6.2 DNS Validation (Critical)

On ESXi:

```
cat /etc/resolv.conf
nslookup <vcenter_fqdn>
nslookup <esxi_fqdn>
```

Ensure:

Forward & reverse DNS working

No stale DNS entries

## 7️⃣ VM Restart Failures
### 7.1 Check Restart Priority

VM → Settings → VM Options → Availability

Restart Priority not set to Disabled

Correct dependency rules (VM-VM rules)

### 7.2 Resource Reservation Conflicts

Check:

CPU / Memory reservations

Large reservations blocking HA placement

## 8️⃣ Log Collection & Analysis
### 8.1 ESXi Logs
```
/var/log/fdm.log
/var/log/hostd.log
/var/log/vmkernel.log
```
### 8.2 vCenter Logs
```
/var/log/vmware/vpxd/vpxd.log
```

Look for:

election

heartbeat

isolation

agent unreachable

## 9️⃣ Recovery & Validation

Confirm:

HA green on all hosts

Master elected

Heartbeat datastores accessible

Perform manual vMotion test

Optional:

Trigger HA test by entering MM on a host (non-prod)

## 🧠 Best Practices (Preventive)

Use 2+ isolation addresses

Separate Management & vMotion VLANs

Monitor storage latency (DAVG < 20ms)

Avoid oversized VM reservations

Keep ESXi versions consistent in cluster

## Knowledge base
Reference link
https://knowledge.broadcom.com/external/article/318936/troubleshooting-vmware-high-availability.html

