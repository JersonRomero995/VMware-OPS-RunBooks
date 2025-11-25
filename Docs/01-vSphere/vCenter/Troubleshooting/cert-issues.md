🛠️ RUN BOOK : vCenter 8.x Certificate Issues
📌 Purpose

Guide to diagnose and resolve certificate-related problems in vCenter 8.x resulting in UI, API, or authentication failures.

🎯 Applies To

🔹 vCenter 8.0, 8.0U1, 8.0U2, 8.0U3
🔹 Embedded PSC (legacy PSC already deprecated)

🚨 1. Common Symptoms

Symptom	Sample Message

Browser warns about certificate	NET::ERR_CERT_DATE_INVALID

Cannot log in after page loads	“Unable to login after verification”

Services fail after reboot/upgrade	vpxd failed to connect

SSL handshake failures	vpxd.certmgr error

Certificate expired	API/UI refuses connection

🔎 2. Diagnosis Steps

A. Check Expiration

```
/usr/lib/vmware-vmca/bin/certool --getcert --server localhost
```

B. Verify SSL Chain
```
Check both UI (443) & VAMI (5480):

openssl s_client -connect <FQDN>:443 -showcerts
openssl s_client -connect <FQDN>:5480 -showcerts
```

C. Check Machine SSL Path
```
ls -l /etc/vmware-ssl/
```

D. Check for Failed Service Startup
```
/usr/lib/vmware-vmon/vmon-cli --status | grep stopped
```

E. Check VMCA Logs
```
less /var/log/vmware/vmware-updatemgr/vmware-updatemgr-syslog.log
less /var/log/vmware/vmcad/*.log
```

🔐 3. Fixes

🟢 Fix 1: Replace Machine SSL Certificate (VMCA Signed)

Use when UI fails or cert is expired, but vCenter is healthy.
```
/usr/lib/vmware-vmca/bin/certificate-manager


Choose:

Option 1 – Replace Machine SSL certificate with VMCA certificate


Then restart services:

service-control --stop --all
service-control --start --all
```

🔵 Fix 2: Regenerate All Certificates

⚠️ Use when multiple services are broken or vmdird fails.
```
/usr/lib/vmware-vmca/bin/certificate-manager


Choose:

Option 4 – Regenerate all certificates using VMCA


Then:

service-control --stop --all
service-control --start --all
```

🟣 Fix 3: Import Custom CA Certificate
```
Use when using enterprise (PKI/MS ADCS) CA certificate.

Run:

/usr/lib/vmware-vmca/bin/certificate-manager


Choose:

Option 2 – Replace Machine SSL certificate with Custom CA certificate


Upload:

root-ca.pem (chain)

signed-cert.pem

private-key.pem

Restart:

service-control --restart vpxd
service-control --restart vsphere-ui
```

🔴 Fix 4: vmdird or SSO Failed Due to Cert Mismatch
```
Symptoms:

Authentication fails

vmdird does not start

Upgrade stuck on identity service

Check logs:

less /var/log/vmware/vmdird/vdcpromo.log
less /var/log/vmware/vmdird/vdcpromo-syslog.log


Fix:

Option 4 – Regenerate all certificates using VMCA
```

🛑 Do NOT revert a snapshot after partial cert change.
Risk: database desync + full identity corruption.

🧪 4. Validation After Fix
Validate	Command/URL
vCenter UI loads	https://FQDN/ui

Certificate chain ok	openssl s_client -connect <FQDN>:443
Services running	/usr/lib/vmware-vmon/vmon-cli --status
Login with SSO	Test administrator@vsphere.local
VAMI works	https://FQDN:5480
🧯 5. Escalation Criteria

Escalate (or restore backup) if:

vmdird logs show DB corruption

No login possible even after full regen

Certificate manager fails to execute

Snapshot rollback already attempted

Recommended action:
📌 Restore from VCSA backup (not snapshot)

Reference links:
https://knowledge.broadcom.com/external/article?legacyId=2112283 