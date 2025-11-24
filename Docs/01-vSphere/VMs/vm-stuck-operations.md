📘 VM Stuck Operations (Power-off, Clone, Delete, Migrate)
🎯 Purpose

Resolve VMs stuck in ongoing tasks like shutdown, cancel, clone, snapshot delete, relocation.

🛑 Symptoms

VM shows in progress task forever

Cannot cancel task

“Another task is already in progress”

Cannot power off/on/clone/migrate

🔍 Troubleshooting Steps
STEP 1 — Get VM ID
vim-cmd vmsvc/getallvms

STEP 2 — Check Executing Tasks
vim-cmd vmsvc/get.tasklist <VM_ID>

STEP 3 — Cancel the Task
vim-cmd vimsvc/task.cancel <TASK_ID>

STEP 4 — Force Kill VM (If Stuck Power State)
esxcli vm process list
esxcli vm process kill --type=force --world-id=<WORLD_ID>

STEP 5 — Check Snapshot Status
vim-cmd vmsvc/snapshot.get <VM_ID>


Consolidate if snapshots exist.

🧹 Prevention Tips

Avoid manual datastore deletions

Avoid killing VM processes if not necessary

Use proper shutdown before maintenance

Monitor snapshot growth with alerts