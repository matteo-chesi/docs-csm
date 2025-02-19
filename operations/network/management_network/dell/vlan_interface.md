# Configure VLAN Interface

The switch also supports classic L3 VLAN interfaces.

## Configuration Commands

Configure the VLAN:

```text
switch(config)# vlan VLAN
```

The default mode of any VLAN is L2 only. To enable L3 functionality, run `no shutdown` on the VLAN:

```text
switch(config)# interface vlan 2
switch(conf-if-vl-2)# no shutdown
```

Show commands to validate functionality:

```text
switch# show interface vlan
```

## Expected Results

1. Administrators can configure the VLAN
2. Administrators can enable the interface and associate it with the VLAN
3. Administrators can create an IP-enabled VLAN interface, and it is up
4. Administrators validate the configuration is correct
5. Administrators can ping from the switch to the client and from the client to the switch

[Back to Index](index.md)
