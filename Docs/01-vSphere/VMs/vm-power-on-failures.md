📘 VM Power-On Failures
🎯 Purpose

Troubleshoot why a VM cannot power on.

🛑 Common Errors

“Insufficient resources”
“Unable to access file <.vmdk>”
“Lock is held”
“Permission denied”
“No space on datastore”

🔍 Troubleshooting Steps
STEP 1 — Check Storage Space
df -h


Delete/Consolidate snapshots if needed.

STEP 2 — Check for Disk Lock

Refer to Runbook (Disk Locking).

STEP 3 — Check Memory Reservation & Admission Control
esxtop


Check %MEMCTL, ballooning, swapping.

Disable reservation temporarily if needed in VM settings.

STEP 4 — Check Corrupted VMX
cat /vmfs/volumes/<datastore>/<VM>/<vmname>.vmx


Search for wrong entries. To fix:

Backup VMX

Re-register VM

vim-cmd solo/registervm /vmfs/volumes/.../<vmname>.vmx

STEP 5 — Check .vswp File Issues

If deletion required:

rm -f <vm>.vswp


⚠ VM must be powered off.

🧹 Prevention Tips

Monitor datastore usage >20% free required

Reduce reservations unless needed

Regular snapshot management

Do not kill hosts during VM migration