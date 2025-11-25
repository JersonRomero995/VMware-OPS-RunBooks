# Run Book: Troubleshooting vCenter Upgrade Issues
Purpose

> This run book provides a step-by-step procedure to diagnose and resolve failures during the upgrade or patching process of VMware vCenter (VCSA) using GUI or CLI (VAMI/ISO Upgrade).

⚠️ Pre-Checks BEFORE Upgrading

Perform before any upgrade attempt:

1. Compatibility & Requirements
Item	Check

- vSphere Compatibility	Verify ESXi hosts HCL vs target vCenter
- Hardware resources	+2 vCPU & +6–8 GB RAM may be needed depending on scale
- Storage	Minimum 20 GB free on /storage/core and /storage/log
- DNS	Forward and Reverse must resolve
- Certificates	Expired certs must be renewed before upgrade
- Backups	Take VCSA snapshot + image-level backup (Veeam if applicable)

```
> CLI Resource Check

> df -h

> du -sh /storage/core

> du -sh /storage/log

> free -m
```

🛠️ COMMON UPGRADE FAILURES & FIXES

🚨 1. Pre-Upgrade Check Fails in Stage 2

Typical Errors

“Not enough space”

“Services failing”

“Health check failed”

Fix Steps

# Clean log bundles
```
rm -rf /storage/log/vmware/*gz
```

# Restart services
```
service-control --stop --all
service-control --start --all
If /storage/core is full
rm -rf /storage/core/*.core
```

🚨 2. vCenter Upgrade Stuck at “Extracting Packages”

Cause: Corrupt ISO, slow storage, or DNS problem.

Fix Steps

# Validate ISO checksum after upload to datastore
sha256sum VMware-vCenter-Server-Appliance-*.iso


Ensure vCenter can resolve itself + PSC (if external):

```
nslookup <FQDN>
nslookup <IP>
ping <FQDN>
```

🚨 3. Stage 2 Fails to Start Services

Errors

vmon error

“Cannot start Service vmdird”

“identity-manager unhealthy”

Fix Steps

```
/usr/lib/vmware-vmon/vmon-cli --status
/usr/lib/vmware-vmon/vmon-cli --restart <service>

Critical Services
vmdird
vmware-vpx
vsphere-client
sso

If vmdird fails, check for expired certificates:

/usr/lib/vmware-vmca/bin/certool --getcert --server localhost
```

🚨 4. Upgrade Fails Due to External PSC

Fix: Migrate PSC to Embedded First:

Use Converge Tool prior to upgrade.

```
cmsso-util converge --platform-services-controller <FQDN>
```

🚨 5. Upgrade Fails Due to Expired Certificates

Fix: Regenerate using VMCA

```
/usr/lib/vmware-vmca/bin/certificate-manager


Choose:

Option 4 – Regenerate SSL certificates and replace VMCA
```

🚨 6. Upgrade Fails Due to DB Issues (PostgreSQL)

Typical Error: “Postgres corruption detected”

Fix Steps

```
/etc/init.d/vmware-vpostgres status
/etc/init.d/vmware-vpostgres restart
```

# Check logs
```
cat /var/log/vmware/vpostgres/*.log
```

🚨 7. Upgrade/Stage 2 Fails After Network Reconfiguration

Error: “VCSA appliance unreachable after upgrade”

Fix Steps

```
Validate eth0 configuration:

ifconfig eth0
cat /etc/systemd/network/10-eth0.network


Restart network:

systemctl restart systemd-networkd
```

📌 POST-UPGRADE VALIDATION
```
Item	Validation
vCenter UI login	https://FQDN/ui

VAMI login	https://FQDN:5480

Cluster and Hosts Connected	All hosts must reconnect
Licensing	Verify licenses applied
Services Health	OK via VAMI
Backup Configs	Backup job re-link if using Veeam

Check services

/usr/lib/vmware-vmon/vmon-cli --status
```

📝 Rollback Procedure

Rollback is valid only if no Stage 2 modifications completed successfully.

If failure occurred BEFORE Stage 2

Revert Snapshot

Restart upgrade

If Stage 2 partially completed

Snapshot rollback not recommended.

Contact VMware GS for DB replay or repair.