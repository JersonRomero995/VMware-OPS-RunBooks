# ping
multiple pings 
```
ping <ip address> -t
```

# netcat
to check communication using ip and ports from the ESXi
```
nc -z <destination-ip> <destination-port>
```

# port block
to review ports block for VM in the ESXI host
```
net-dvs -l | grep -E "port |port.block|volatile.vlan|volatile.status"
```

# Net commands
Identify the port IDs for all connected interfaces on the ESXi host
```
net-stats -l
```

# esxcfg commands
to Identify theswitch name on the host
```
esxcfg-vswitch -l
```