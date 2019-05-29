# Bug: tap0 interface state is DOWN (deleted VM's manually)

## Summary
Don't delete virtual machines manually. Just use `vagrant destroy <node_name>"`.

## What happened
I deleted the virtual machines from the `VirtualBox VMs` folder. This can mess up associations with the file names. In my case, prevented the state of the `tap0` interface to go `UP`.
For example, when typing `ip addr`:
``` 
8: tap0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state DOWN...
```
https://medium.com/@philw_/deleting-inaccessible-vms-from-the-virtualbox-gui-via-the-command-line-580a85845360
