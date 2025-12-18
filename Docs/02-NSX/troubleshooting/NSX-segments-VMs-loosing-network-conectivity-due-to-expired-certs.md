# Impact

## NSX UI/API instability

Potential impact to control plane operations (DFW, routing changes, policy updates)

Data plane usually continues forwarding traffic (verify)

## 1. Initial Triage
### 1.1 Scope the Issue

Is this single NSX Manager node or entire cluster?

Any recent changes?

Certificates

NTP/DNS changes

Network/firewall changes

vCenter upgrade

Disk expansion or snapshot activity

##1.2 Confirm NSX Version

From UI or CLI:
```
get version
```
## 2. Health Checks
### 2.1 NSX UI Health

Navigate to:

System → Appliances → NSX Manager

System → Fabric → Nodes → Managers


Check:

Cluster Status

Node Status

Services Status

### 2.2 NSX Manager CLI Health Checks

SSH into each NSX Manager node.
```
get cluster status
get managers
get service all
get node status
```

Expected:

Cluster status: STABLE

All managers: UP

Critical services: running

### 2.3 Messaging Bus / Control Plane
```
get logical-routers
get transport-nodes
```

If cluster-related commands hang or fail → suspect messaging bus or service-level issue.

## 3. Common Root Causes & Validation
### 3.1 NTP (Most Common)
```
get ntp-server
get ntp-status
```

Validate:

Same NTP servers on all nodes

Time offset < 5 seconds

Fix if needed:
```
set ntp-server <ntp_ip>
restart service ntpd
```
### 3.2 DNS Resolution
```
get dns-servers
nslookup <manager_fqdn>
hostname -f
```

Validate:

Forward and reverse DNS match

No hostname changes after deployment

### 3.3 Certificate Issues
```
get certificate api
get certificate cluster
```

Look for:

Expired or near-expiry certs

Trust mismatch between nodes

### 3.4 Disk Space
df -h


Pay attention to:

/

/var/log

/image

If full:

Remove old logs

Rotate logs

Extend disk if required

### 3.5 Network / MTU / Firewall

Validate:

Manager-to-manager connectivity

Required ports open (TCP 5671, 443, 123, etc.)

MTU consistency (especially if overlay is involved)

Test:

vmkping <peer_manager_ip>

## 4. Log Collection (Before Any Restart)
### 4.1 Generate Support Bundle

From NSX UI:

System → Support → Support Bundles

### 4.2 Key Logs (CLI)
```
/var/log/nsxapi.log
/var/log/syslog
/var/log/proton/nsx-manager.log
/var/log/corfu/*
```
## 5. Remediation Steps

⚠️ Always start with root cause (NTP/DNS/Disk). Do NOT blindly restart all services.

### 5.1 Restart Individual Services (If Needed)
```
restart service nsx-manager
restart service proton
```

Check status after:
```
get service all
```
### 5.2 Restart Management Services (Last Resort)
```
restart service all
```

Monitor cluster status closely.

``` 
In this case the issue was still after renewing the certificates with the carr script
https://knowledge.broadcom.com/external/article/369034 

Then we realized that the edges were having issues, carr is suppose to update those certificates as well but if connection is already lost the nsx manager won't be able to communicate with the edges therefore not able to renew the certificates, in this case those certificates need to be updated directly in the edge
```


## 6.1 Edge Health – UI Validation

Navigate to:

System → Fabric → Nodes → Edge Transport Nodes


Check:

Edge status: UP

Management connectivity: Connected

Alarms mentioning:

MPA disconnected

Certificate trust issues

Edge cannot connect to manager

Also check:

Networking → Tier-0 / Tier-1 → Interfaces / Routing


If UI is slow or partially broken, proceed with CLI checks.

## 6.2 Edge CLI – Basic Health Checks

SSH into each NSX Edge Node.

### 6.2.1 Node & Service Status
```
get node status
get service all
```

Validate:

nsx-edge, nsx-mpa, proton services are running

Node state: UP

### 6.2.2 MPA (Management Plane Agent) Status
```
get mpa status
```

Common bad states:

DISCONNECTED

CERTIFICATE_ERROR

AUTHENTICATION_FAILED

If MPA is not CONNECTED, continue with certificate checks.

## 6.3 Edge Certificate Validation
### 6.3.1 List Edge Certificates
```
get certificate
```

Focus on:

Edge node certificate

API / trust certificates

Expiration dates

Issuer consistency with NSX Managers

### 6.3.2 Validate Manager Trust from Edge
```
get certificate api
```

Look for:

Expired Manager certs

Trust chain mismatch

Missing CA certs

### 6.3.3 Verify Edge ↔ Manager Connectivity
```
get managers
```

Expected:

NSX Managers listed as UP

No TLS or auth errors

## 6.4 Common Edge Certificate Failure Scenarios
```
Scenario	Symptom
Edge cert expired	MPA disconnected
CA renewed but Edge not updated	TLS handshake failures
Partial CARR execution	Managers healthy, Edges broken
Time drift on Edge	Cert appears invalid
Edge truststore not refreshed	Persistent MPA failure
```
##6 .5 Edge Remediation Steps

⚠️ Perform during maintenance window if possible

### 6.5.1 Restart MPA Service
```
restart service nsx-mpa
```

Recheck:

```
get mpa status
```

### 6.5.2 Restart Edge Services (If Needed)
```
restart service nsx-edge
restart service proton
```
### 6.5.3 Force Certificate Refresh (If Managers Are Healthy)

From NSX Manager UI:

System → Certificates → Refresh Edge Certificates


Or re-push configuration by:

Disconnecting and reconnecting Edge from Fabric (last resort)

## 6.6 Logs to Collect (Edges)

Before escalation or redeploy:

```
/var/log/nsx-edge.log
/var/log/mpa.log
/var/log/syslog
```

Also collect:

Edge support bundle from NSX UI

## 6.7 Post-Fix Validation (Edges)
### 6.7.1 MPA Connectivity
```
get mpa status
```

Expected:

CONNECTED

### 6.7.2 Fabric Status
```
get transport-nodes
```

Edges should show:

UP

No cert-related alarms