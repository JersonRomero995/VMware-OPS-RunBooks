🔐 RUN BOOK: vCenter 8.x UI Login Failures
📌 Purpose

Standardized troubleshooting workflow for failures logging into the vCenter UI at:

https://<FQDN>/ui

🎯 Applies To

✔️ vCenter 8.0, 8.0U1, 8.0U2, 8.0U3

✔️ Embedded PSC (no external PSC support)

🚨 1. Common Symptoms

Symptom	Example Message

Login page loads but authentication fails	“Unable to login after verification”

Blank loading screen / stuck	Spinning icon / page reloads

503 errors	Service Unavailable

SSL warnings	NET::ERR_CERT_DATE_INVALID

SSO unavailable	Identity source unreachable

🔎 2. Quick Diagnostics
A. Validate Connectivity

- nslookup <FQDN>
- ping <FQDN>
- hostname -f

B. Validate Ports

nc -zv <FQDN> 443

C. Check Key Services

/usr/lib/vmware-vmon/vmon-cli --status | grep -E 'stopped|not'


Critical services:

Service	Function:
- vsphere-ui	UI frontend
- vmware-stsd	Security token
- vmdird	Identity / SSO
- vmware-vapi-endpoint	Back-end API
- vmware-vpxd	vCenter Core

D. Check System Resources

Low storage or RAM causes login failures.

- df -h
- free -m

🐞 3. Check Logs for Login Failures

UI Logs

less /var/log/vmware/vsphere-ui/logs/vsphere_client_virgo.log

Authentication / SSO

-less /var/log/vmware/sso/*.log
-less /var/log/vmware/vmdird/vdcpromo.log

VAPI Backend

-less /var/log/vmware/vapi/*.log

🛠️ 4. Fixes

🟢 Fix 1: Restart Authentication Services

- /usr/lib/vmware-vmon/vmon-cli --restart vmware-stsd
- /usr/lib/vmware-vmon/vmon-cli --restart vmdird

🔵 Fix 2: Restart UI Components

- /usr/lib/vmware-vmon/vmon-cli --restart vsphere-ui
- /usr/lib/vmware-vmon/vmon-cli --restart vsphere-client

🟣 Fix 3: Restart API Backend
-/usr/lib/vmware-vmon/vmon-cli --restart vmware-vapi-endpoint

🔴 Fix 4: Fix Invalid/Expired Certificates Affecting Login

Run:

/usr/lib/vmware-vmca/bin/certool --getcert --server localhost


If expired:

/usr/lib/vmware-vmca/bin/certificate-manager


Choose:

Option 1 – Replace Machine SSL certificate with VMCA certificate


Then restart:

- service-control --stop --all
- service-control --start --all

🟡 Fix 5: Identity Source Authentication Errors

Symptoms:

Domain users cannot log in

Only administrator@vsphere.local works

🧰 Steps

Log in using:

administrator@vsphere.local

Go to:
Administration → Single Sign On → Configuration → Identity Sources

Validate:

Domain controller FQDN reachable

Bind user credentials valid

Port 389/636 reachable

📌 CLI DC Connectivity Check

- nc -zv <domain-controller> 389
- nc -zv <domain-controller> 636

⚠️ Fix 6: Low Disk/Storage

Login may fail if /storage/log or /storage/db is full.

Check:

df -h


Clean logs (safe):

rm -rf /storage/log/*.gz
rm -rf /storage/log/vmware/*gz

🔧 Fix 7: Browser Cache or Old Certificates

Clear browser cache

Try incognito mode

Try another browser

Clear SSL state in OS (Windows: inetcpl.cpl → Content → Clear SSL)

🧪 5. Validation

Check	Expected

https://FQDN/ui
 loads	Login screen displays
Login works with SSO	administrator@vsphere.local
 works

Services running	vmon-cli --status no failures

Back-end API	No errors in /var/log/vmware/vapi/

SSL healthy	openssl s_client -connect <FQDN>:443 OK

🧯 6. Escalation Criteria

Escalate or restore backup if:

vmdird database corruption detected

Certificate corruption unresolved after regeneration

No login possible even after fixing SSO + Back-end services

Recommended action:
📌 Restore from VCSA backup taken before issue occurred.