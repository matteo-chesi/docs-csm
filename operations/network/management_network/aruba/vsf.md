# Virtual Switching Framework (VSF) - 6300 Only

Virtual Switching Framework (VSF) defines a virtual switch comprised of multiple individual physical switches, inter-connected through standard Ethernet links. These physical switches will operate with one control plane, thereby visible to the peers as a virtual switch stack.

Within the stack, one switch is the "Master" switch, which runs all the control plane software and manages the ASICs of all the stack members. A second switch can be configured as the "Standby" switch, which will take over as master if the master fails.

Each stack member must have a unique member number. While deploying a stack, the user must ensure that each member has a distinct member number by renumbering the switches to desired member numbers. Stack formation will fail if there is a member number conflict.

A primary member will become master under normal circumstances (no member / link failures). The secondary member will become standby member under normal circumstances.

The primary member is member number `1`. This is not configurable. Also note that `1` is the default member number. A factory-default switch boots up as a VSF-enabled switch, with member number "1". It behaves as a 1-member stack, of which it is master.

The secondary member number is user configurable, and there is no default secondary member. It is strongly recommended that the customer configure a secondary member in the stack, because a stack with a standby offers resiliency and high-availability.

Other than the primary and secondary members, no members can ever become master / standby of the stack.

## Configuration Commands

Create a VSF member:

```text
switch(config)# vsf member <ID>
switch(vsf-member)# link <ID> <IFACE-RANGE>
```

Show commands to validate functionality:

```text
switch# show vsf <brief|configuration|status>
```

## Example Output

```text
switch# show vsf topology
          Stby     Master
CPU Utilization
Memory Utilization : 15%
VSF link 1         : Down
VSF link 2         : Down
+---+    +---+    +---+
| 3 |1==2| 2 |1==1| 1 |
+---+    +---+    +---+
```

[Back to Index](../index.md)
