🛑 RUN BOOk: VAMI (https://FQDN:5480
) Not Loading on vCenter 8.x

📌 Purpose

Troubleshoot failures when accessing the vCenter Appliance Management Interface (VAMI) at:

https://<FQDN>:5480

🎯 Applies To

✔️ vCenter 8.0, 8.0U1, 8.0U2, 8.0U3
✔️ Embedded PSC only

🚨 1. Symptoms

Symptom	Message/Behavior

- Browser shows connection refused	ERR_CONNECTION_REFUSED
- VAMI page blank / keeps loading	Spinning / No login
- SSL error	NET::ERR_CERT_DATE_INVALID
- Redirect to error	“Appliance management service unavailable”
- vCenter UI works but VAMI does not	Only port 5480 affected

🔎 2. Initial Diagnostics

A. Test Port Availability (5480)

Run on workstation:

nc -zv <FQDN> 5480

B. Check DNS Resolution
- nslookup <FQDN>
- hostname -f

C. Verify VAMI Service Status

systemctl status applmgmt

D. Check System Resources

Low storage or memory prevents VAMI from loading.

- df -h
- free -m

🐞 3. Collect Logs

VAMI Service Logs

less /var/log/vmware/applmgmt/applmgmt.log

Appliance Configuration

less /var/log/vmware/applmgmt/*.log

Check if Reverse Proxy is failing

less /var/log/vmware/rhttpproxy/*log

🛠️ 4. Fixes

🟢 Fix 1: Restart Appliance Management Service

- systemctl restart applmgmt


Validate:

- systemctl status applmgmt

🔵 Fix 2: Restart Reverse Proxy

Only if 5480 reachable but blank page loads.

service-control --restart rhttpproxy

🟣 Fix 3: Fix Expired/Invalid VAMI Certificate

Check expiration:

openssl s_client -connect <FQDN>:5480 -showcerts


If expired → replace Machine SSL certificate:

/usr/lib/vmware-vmca/bin/certificate-manager


Select:

Option 1 – Replace Machine SSL certificate with VMCA certificate


Then restart:

- service-control --stop --all
- service-control --start --all

🔧 Fix 4: Low Disk Space Blocks VAMI Startup

Check:

df -h


If /storage/log or /storage/core is full:

- rm -rf /storage/log/*.gz
- rm -rf /storage/log/vmware/*gz
- rm -rf /storage/core/*.core


Restart service:

systemctl restart applmgmt

🔴 Fix 5: VAMI Broken After Failed Upgrade/Stage 2

If upgrade failed and VAMI won’t load:

Restore services:

- service-control --stop --all
- service-control --start --all


If still broken, restore from backup:
📌 Restore VCSA backup taken before upgrade

⚠️ Do NOT revert snapshots after failed upgrade. Risk of DB corruption.

🟡 Fix 6: Browser/Cache Issue

Use incognito mode

Try different browser

Clear OS SSL cache (Windows):

inetcpl.cpl → Content → Clear SSL State

🧪 5. Validation Checklist
- Check	Expected Result
- Port 5480 reachable	nc -zv <FQDN> 5480 succeeds
- VAMI loads in browser	Login page displayed
- Certificate chain valid	openssl s_client -connect <FQDN>:5480 no errors
- Service active	systemctl status applmgmt = running
- Proxy stable	No errors in /rhttpproxy/ logs

🧯 6. Escalation Criteria

Escalate/restore backup if:

applmgmt service fails repeatedly after restart

rhttpproxy is corrupted

Failed upgrade damaged appliance config

Certificate regeneration fails

🔄 Recommended Action: Restore last full VCSA backup
❌ Do not revert snapshots.