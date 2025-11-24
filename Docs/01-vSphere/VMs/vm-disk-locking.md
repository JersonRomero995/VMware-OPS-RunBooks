📘 VM Disk Locking Issue
🎯 Purpose

Troubleshoot situations where a VM cannot power on, consolidate disks, migrate, or access VMDKs due to lock conflicts.

🛑 Common Errors

“Failed to lock the file”

“Cannot open the disk ‘xxxxx.vmdk’ or one of the snapshot disks is already in use”

“Cannot power on virtual machine: file is locked”

Consolidation timeout

🔍 Step-by-Step Troubleshooting
STEP 1 — Identify Lock Owner

SSH to ESXi suspected host:

vmfsfilelockinfo -p /vmfs/volumes/<datastore>/<VM>/<vmname>.vmdk


👀 Review owner info: MAC, IP, Host UUID.

STEP 2 — Cross-check the Host

Confirm if the host is alive and part of vCenter:

esxcli system hostname get
esxcli network ip interface ipv4 get


If host offline → that stale lock must be force cleared by reboot.

STEP 3 — Check for Orphaned VM or Suspended State
vim-cmd vmsvc/getallvms
vim-cmd vmsvc/power.getstate <VM_ID>


If suspended:

vim-cmd vmsvc/power.off <VM_ID>


Delete .vmss and .vswp if needed and safe.

STEP 4 — Verify NFS/Storage APD/PDL (if remote storage)

Check logs:

grep -i "device.*lock" /var/log/vmkernel.log
grep -i "APD" /var/log/vmkernel.log


If APD/PDL → fix storage connectivity.

STEP 5 — Restart Host Agents (Safe)
/etc/init.d/hostd restart
/etc/init.d/vpxa restart


⚠ Do NOT restart vmkernel storage services as it may cause interruptions.

STEP 6 — Last Resort: Reboot Lock Owner Host

Confirm no running VMs affected → reboot host.

🧹 Prevention Tips

Do not abrupt power off hosts

Ensure datastore heartbeat available

Avoid stale snapshots

Use proper NFS/Storage timeouts


Reference links:
https://knowledge.broadcom.com/external/article/418256/virtual-machine-backups-fail-due-to-lock.html 
https://knowledge.broadcom.com/external/article/314365/investigating-virtual-machine-file-locks.html 
https://knowledge.broadcom.com/external/article/393030 