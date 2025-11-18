# 0. Scope & Assumptions

This runbook applies when:
One or more datastores show as “inactive”, “inaccessible”, or “disconnected” after:
Host shutdown/startup
Storage array restart
Site / power outage
Environment: vSphere 7 / 8, ESXi hosts, VMFS/NFS/iSCSI/FC.

# 1. Quick Triage in vSphere Client

- Identify impact

Are all datastores down or only some?
Are they down on all hosts or only specific hosts?

- Check datastore status

Go to vCenter → Storage → Datastores.
Note:
Icon: grey/inaccessible?
Type: VMFS / NFS / vVOL / vSAN.
Which hosts still see it (if any).
Check host-level visibility in UI

Select an affected ESXi host → Configure → Storage → Devices.

See if the underlying LUN/device is visible or missing.
If device is missing for all LUNs → suspect storage / fabric / network.
If device is visible but datastore is inaccessible → suspect filesystem / mount issue.

From this you decide which branch: Block storage (VMFS) or Network storage (NFS/iSCSI).

# 2. On the ESXi Host – General Checks

SSH to an affected host (or use DCUI → ESXi Shell).

# 2.1. List filesystems and status

esxcli storage filesystem list

Look for the datastore in question:
Check Mount Point, Type (VMFS, NFS), and Mounted (true/false).

If not listed at all → device not visible or not mounted.
If listed but Mounted: false → focus on mount issues.

# 2.2. List storage devices
esxcli storage core device list

Confirm the device corresponding to the datastore exists.
If it’s missing here, ESXi does not see the LUN/disk.

# 2.3. List storage paths
esxcli storage core path list

Check if paths are:
Active, Standby, Dead, or Off.
If all paths are Dead/Off for that device → storage/fabric issue.

# 2.4. List HBAs
esxcli storage core adapter list

Confirm HBAs (vmhba adapters) are Online.
If any HBA is Off or missing after shutdown → check hardware/fabric and reboot/enable as needed.

# 3. Block Storage (VMFS on FC/iSCSI/SAS)
# 3.1. Rescan HBAs

From one affected ESXi host:

esxcli storage core adapter rescan --all
esxcli storage core device list
esxcli storage filesystem list

Then check if datastore appears or becomes mounted.
You can also trigger rescan from UI:

Host → Actions → Storage → Rescan Storage (or “Rescan HBAs”).

# 3.2. Check for APD/PDL in logs
grep -i "APD" /var/log/vmkernel.log
grep -i "PDL" /var/log/vmkernel.log


APD (All Paths Down) → ESXi cannot reach storage, usually fabric/network/array issue.
PDL (Permanent Device Loss) → array saying “LUN is gone” (unpresented/failed).

If APD/PDL is present:
Validate with storage team if LUNs are properly presented and online to the host WWPNs/iSCSI IQNs.

# 3.3. Verify VMFS volumes
esxcli storage vmfs extent list

Confirm your datastore name and backing device.
If device is there but datastore missing in filesystem list, you may need to manually mount/restore.

# 3.4. For iSCSI-based VMFS

Check iSCSI adapters

esxcli iscsi adapter list
esxcli iscsi adapter target portal list -A vmhbaXX
esxcli iscsi adapter target list -A vmhbaXX

Check network connectivity from vmk to target
Find vmk used for iSCSI:

esxcfg-vmknic -l

Ping target IP from that vmk:

vmkping <iSCSI_target_IP>


If vmkping fails → check vSwitch, VLAN, physical switch ports, and routing.
If vmkping works but LUN not visible → storage-side config (target, LUN masking, CHAP, etc.).

# 4. Network Storage – NFS Datastores

If the datastore type is NFS:

# 4.1. List NFS mounts
esxcli storage nfs list

Check:
Host (NFS server IP/FQDN)
Share (export path)
Accessible (true/false)

If missing there → mount was removed/never re-added on the host.

# 4.2. Test network connectivity
vmkping <NFS_server_IP>

If vmkping fails:
Check:
vmkernel NIC used for NFS:

esxcfg-vmknic -l

vSwitch/portgroup/VLAN
Physical switch ports
Routing:

esxcfg-route -l

# 4.3. Try remount NFS

If network is fine, but NFS datastore is inaccessible:
Remove stale entry (if needed):
esxcli storage nfs remove -v <datastore_name>

Add it again:
esxcli storage nfs add \
  -H <NFS_server_IP_or_FQDN> \
  -s <export_path> \
  -v <datastore_name>

Then verify:
esxcli storage nfs list
esxcli storage filesystem list

# 5. Host Connectivity & Fabric Checks

If multiple datastores are down on one host only:
Check physical NICs:

esxcli network nic list

Ensure links are Up.
Check vmkernel interfaces (for iSCSI/NFS):

esxcfg-vmknic -l

Check routing:
esxcfg-route -l

Validate with network/storage teams:
Zoning (FC)
VLANs & trunking (iSCSI/NFS)
LACP or port-channel status

# 6. vCenter vs Host: Is it Just an Inventory Issue?

Sometimes the datastore is visible on the host, but vCenter still shows it as “disconnected”.
In vSphere Client:
Compare host view vs vCenter view.
If host sees the datastore (CLI+UI) and vCenter doesn’t:
Refresh vCenter inventory, or disconnect/reconnect host (CAREFULLY, after ensuring no running production impact).
Check hostd & vpxa logs for inventory sync issues:

tail -n 100 /var/log/hostd.log
tail -n 100 /var/log/vpxa.log

# 7. Logs to Collect for Escalation

If basic steps do not resolve:
From an affected host, grab:
vm-support

Or collect a host support bundle from:
vSphere Client → Host → Actions → Generate support bundle.
Note timestamps when:
Hosts/shutdown happened
Storage came back
You saw the issue
So you can correlate in vmkernel.log, vobd.log, hostd.log.