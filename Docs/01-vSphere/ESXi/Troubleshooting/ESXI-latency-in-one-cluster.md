# Runbook: ESXi Host Latency Only Inside a Specific Cluster (vSphere 8)

## Scenario:
One ESXi host experiences high latency only when it is inside a specific cluster. When the same host is moved (Disconnected → Removed → Added) to another cluster in the same vCenter, the latency disappears.

1.2 Validate ESXi Host Health

SSH → run:

esxcli hardware pci list
esxcli hardware cpuid get
esxcli hardware ipmi sel get
esxtop (press d for disk, n for network)
