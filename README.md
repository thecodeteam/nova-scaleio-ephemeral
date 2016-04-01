# ScaleIO ephemeral backend for Nova
This project provides code patches for Nova to use EMC ScaleIO as ephemeral storage backend.

## Installation
Figure out installed version of Nova (e.g. 2015.1.2). Make appropriate diff file available on a compute node dedicated to patching. In the node console type:
```
$ patch -p 2 -i \<path to diff file\> -d \<path to installed nova\>
```
E.g.:
```
$ patch -p 2 -i ~/2015.1.2.diff -d /usr/lib/python2.7/dist-packages/nova
```
It is recommended to run this command previously with --dry-run option to make sure the patch will be applied correctly. The output should look like this:
```
checking file nova/virt/libvirt/driver.py
checking file nova/virt/libvirt/imagebackend.py
checking file nova/virt/libvirt/sio_utils.py
checking file nova/virt/libvirt/utils.py
```
But not like that:
```
checking file nova/virt/libvirt/driver.py
Hunk #6 succeeded at 1367 with fuzz 1 (offset -13 lines).
Hunk #7 FAILED at 1403.
1 out of 14 hunks FAILED
checking file nova/virt/libvirt/imagebackend.py
Hunk #3 succeeded at 870 (offset -9 lines).
Hunk #4 succeeded at 980 (offset -9 lines).
checking file nova/virt/libvirt/sio_utils.py
checking file nova/virt/libvirt/utils.py
Hunk #2 succeeded at 476 with fuzz 2 (offset -16 lines).
```
If the output contains failures or fuzzes, the patch does not suit installed Nova.

After applying a patch configure (see the next chapter) and restart Nova compute service.

## Configuration
Change Nova compute config to let Nova use ScaleIO as ephemeral backend. Following options are mandatory:
```
[libvirt]
images_type = sio

[scaleio]
rest_server_ip = <ScaleIO Gateway IP>
rest_server_username = <ScaleIO Gateway user>
rest_server_password = <ScaleIO Gateway user password>
protection_domain_name = <ScaleIO protection domain to be used for ephemeral disks>
storage_pool_name = <ScaleIO pool name to be used for ephemeral disks>
default_sdcguid = <ScaleIO SDC guid of compute host>
```
To get compute host's SDC guid you can use:
```
$ /bin/emc/scaleio/drv_cfg --query_guid
```
It is important to specify default_sdcguid parameter in upper case.

Other options (for live migration, etc) depend on your installation but do not depend on ScaleIO ephemeral backend.

## Flavor restriction
Since ScaleIO allows volume sizes to be multiples of 8 GB, used flavors must have appropriate disk, ephemeral, and swap sizes. So either update existing flavors, or create new ones. Otherwise any attempt to boot an instance with ephemeral device(s) will be finished with 'No valid host was found' error.

Moreover this restriction affects ephemeral and swap requests by block device mapping of images or cli calls.

## Limitations
* Thick volume provisioning is supported only at the moment.
* Impossible to use several ScaleIO storage pools from one compute node.
