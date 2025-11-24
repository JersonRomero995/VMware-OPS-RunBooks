# issue
An error occurred while consolidating disks: The operation failed. Consolidation falied for disk node 'scsi0:0': The operation failed. Failed to get copy progress while consolidating disks from 'vmfs/volumes/#############/<vm name>'. Failed to copy source (vmfs/volumes/#############/<vm name>) to destination (vmfs/volumes/#############/<vm name>): Timeout


🧠 Root Cause Summary

This error usually occurs when VM snapshot VMDKs cannot be merged due to:

✔ Locked VMDK files
✔ Stuck async I/O or stale processes
✔ Storage latency / datastore connectivity issues
✔ Lack of space on the datastore
✔ Snapshot chain corruption
✔ vSphere replication, backup, or third-party tools working on the disk

🛠️ Troubleshooting Procedure
1️⃣ Validate Disk Space

Open datastore browser and verify enough free space:

Need at least same size as the largest snapshot delta file.

If < 10–15% free → extend datastore or migrate VM to a larger datastore.

If low disk space →
Move VM to another datastore:

Storage vMotion VM → then try Consolidate again.

2️⃣ Check Datastore / Storage Latency (likely cause)

Open ESXTOP on ESXi hosting the VM:

ssh root@esxi-host
esxtop
u   (disks)


⚠ If DAVG > 20ms, or you see >100ms spikes → storage issue.

➡ Wait until storage stabilizes OR vMotion the VM to another datastore and retry consolidation.

3️⃣ Identify VMDK Locks

Run:

vmkfstools -D /vmfs/volumes/<datastore>/<VM>/<diskname>.vmdk


If lock shows a MAC different from the host → check which host:

esxcli storage core device list | grep <Lock info>


➡ If a stale lock exists → reboot host holding the lock OR restart host services:

/etc/init.d/hostd restart
/etc/init.d/vpxa restart


⚠ Only restart on the host holding the lock. Do NOT restart hosts blindly.

4️⃣ Check if Backup/DR Job Stuck

Check if tools such as Veeam, Commvault, Dell AppSync, SRM, vSphere Replication are holding the snapshot.

Veeam - SSH to proxy/hypervisor and kill stale tasks:

ps -c | grep veeam
kill -9 <PID>

5️⃣ Resume Stuck Snapshot Consolidation Using CLI

Navigate to VM folder:

cd /vmfs/volumes/<datastore>/<VM>/


Check snapshot chain:

vmkfstools -q <diskname>.vmdk


Then force consolidate manually:

vmkfstools -i <source-delta.vmdk> <target.vmdk> -d thin


💡 Use thin to avoid space explosion, unless required otherwise.

After clone completes, reattach VMDK to VM.

6️⃣ Use VMware “Consolidate Helper Workspace” Fix

If consolidation shows helper snapshot:

vim-cmd vmsvc/getallvms
vim-cmd vmsvc/snapshot.get <VMID>
vim-cmd vmsvc/snapshot.removeall <VMID>

7️⃣ Repair Snapshot Chain Corruption

If snapshot descriptor is broken, fix using:

cat <delta.vmdk> | grep parentFileNameHint


If incorrect path → manually edit .vmdk descriptor:

vi <delta.vmdk>
parentFileNameHint="<correct parent>.vmdk"


Save and retry consolidation.

8️⃣ If Everything Fails – Cold Clone Recovery

⚠️ Only if VM downtime is allowed

Power Off VM

Clone the disk using vmkfstools:

vmkfstools -i <broken-delta.vmdk> <newdisk.vmdk> -d thin


Create a new VM and attach the cloned disk

🧪 Verification Steps

Once done, confirm:

Check	Expected
Datastore free space	> 10–15%
VM consolidate action	Succeeds without warnings
No active snapshots	Consolidated = Yes
Application consistency	VM boots/apps ok


🧹 Post-Fix Preventive Measures

✔ Set alarms for snapshot >72 hours
✔ Monitor storage latency via vROps / Aria / esxtop
✔ Implement backup job timeout tuning
✔ Avoid keeping quiesced snapshots during heavy I/O jobs

kbs:
https://knowledge.broadcom.com/external/article/414348/vm-consolidation-failed-with-the-error-f.html 


