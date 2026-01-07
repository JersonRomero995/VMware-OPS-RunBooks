# 1. Initial Information Gathering

Before changes, collect baseline data.

## 1.1 Confirm the scope

Is the issue affecting one VM, multiple VMs, or the entire port group/VLAN?

Is the issue intermittent or constant?

## 1.2 Gather VM details

Record:

ESXi host name

vCenter version

VM hardware version

vNIC type (VMXNET3 / E1000)

Port group name

VLAN ID

IP, MAC address

# 2. Basic VM-Level Verification
## 2.1 Guest OS checks

Inside the VM:

Check IP configuration:

Windows:
```
ipconfig /all
```

Linux:
```
ip addr show
```

Check default gateway

Check DNS settings

Check firewall rules

## 2.2 Tools to test connectivity

Run:
```
ping <gateway>

ping <DNS server>

tracert (Windows) / traceroute (Linux)

nslookup for DNS issues

If cannot ping the gateway, suspect host/port group/VLAN.
```

https://blogs.vmware.com/cloud-foundation/2018/12/20/esxi-network-troubleshooting-tools/ 

# 3. vCenter / VM Network Adapter Verification
## 3.1 Validate VM Network Adapter

Power on the VM → Edit Settings

Correct Network Adapter connected

Correct Port Group

Adapter type: VMXNET3 recommended

## 3.2 Check MAC & vNIC

Ensure no duplicate MACs

Check if vNIC is connected and “Connected at power on” is checked.

## 3.3 Verify port group assignment

Confirm the VM is using the intended:

vSwitch / vDS

Port group

VLAN ID

# 4. Switch/Port Group/VLAN Verification
## 4.1 Port group configuration

vCenter → Networking → Port Group:

Verify correct VLAN ID

Check security settings:

Promiscuous Mode

MAC Address Changes

Forged Transmits

If MAC spoofing or load balancer is used → ensure settings are not blocking.

## 4.2 dvPort status (if using vDS)

Select VM → Manage dvPort

Look for:

Blocked state

Health status

MTU mismatches

# 5. ESXi Host-Level Troubleshooting
## 5.1 Validate host physical adapters (vmnic)

SSH to ESXi:
```
esxcli network nic list
```

Check:

Link state

Speed/duplex

Driver/firmware mismatches

## 5.2 Check uplink failures
```
esxcli network nic get -n vmnicX
```
## 5.3 Check vSwitch configuration

Standard Switch:
```
esxcli network vswitch standard list
```

vDS:
```
esxcli network vswitch dvs vmware list
```
## 5.4 Check port status assigned to VM

Find the VM’s vNIC port ID:
```
esxcli network vm list
esxcli network vm port list -w <WorldID>
```

Validate:

Team/Uplink assignment

VLAN

Link State

# 6. VLAN and Physical Network Troubleshooting
## 6.1 Check VLAN is allowed on trunk

Confirm with network team:

Switchport trunk is allowing VLAN

No STP blocking

No disabled ports

## 6.2 MTU Mismatch

Verify MTU:
```
esxcfg-vmknic -l
esxcfg-vswitch -l
```

Ping test:
```
ping -s 8972 -d <destination>
```
# 7. NSX-Specific Checks (if applicable)
## 7.1 Verify Logical Switch / Segment status

NSX Manager → Networking → Segments

Confirm:

Segment is up

Transport Zone matches host

## 7.2 Verify Host Transport Node

Check:

TEP IPs

Tunnel status

VIBs installed

On ESXi:
```
get host-switch-state
```

(For NSX-T)

## 7.3 Check Edge/DFW

DFW rules blocking?

Real-time packet logs:

start firewall debug

# 8. Packet Capture Analysis
## 8.1 ESXi packet capture

Standard Switch:
```
tcpdump-uw -i vmnicX
```
https://knowledge.broadcom.com/external/article?legacyId=2051814 --> packet capture KB

Some captures example:
```
pktcap-uw --uplink vmnicX --capture UplinkSndKernel,UplinkRcvKernel -s 160 --mac <mac> -0 - | tcpdump-uw -enr -
```
vDS (using port ID):

vsish -e get /net/portsets/<switch>/ports/<portID>/stats

## 8.2 Guest OS capture

Use:

Wireshark

tcpdump (Linux)

# 9. Logs Review
## 9.1 ESXi logs

Check:
```
/var/log/vmkernel.log

/var/log/vmkwarning.log

/var/log/vobd.log
```
Search for:
```
grep -i "net" /var/log/vmkernel.log
grep -i "link" /var/log/vmkernel.log
grep -i vlan /var/log/vmkernel.log
```
# 10. Remediation Actions
## 10.1 Reset vNIC

Disconnect / reconnect network adapter

Remove adapter → add a new VMXNET3

## 10.2 Migrate VM to another host

Test if host hardware/uplinks are the problem.

## 10.3 Restart Management Agents

If port groups are not responding:
```
services.sh restart
```

or safer:
```
/etc/init.d/hostd restart
/etc/init.d/vpxa restart
```
# 10.4 Reapply port group

Change to a different port group → revert back.

# 10.5 vMotion VM to a healthy host

If network recovers → host issue.
```

# Note
From captures if you get the traffic from the uplink and for instance you see several ARP request leaving and no response getting into the uplink the affected VM or VMs this is most likely a physical en issue.

You need to review and make sure the vlan ID is the same in the port group in the ESXI host compare to the physical nic connected to the uplink vnic, or review if there is any firewall rule blocking the traffic.
