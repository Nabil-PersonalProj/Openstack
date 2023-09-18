For deploying cinder, you need to have already deployed your openstack cloud
we need to prepare the storage node first for cinder. Ensure that there is a disk for cinder.
1. create a physical volume
```
$pvcreate /dev/sdb ("sdb" will be the disk you want to use)
```

2. Create a volume group
```
$vgcreate cinder-volumes /dev/sdb
```

3. edit the /etc/modules file
```
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
configfs
```

4. rebuild initramfs
```
update-initramfs -u
```

5. stop and disable open-iscsi and iscsid
```
systemctl stop open-iscsi
systemctl disable open-iscsi
```
```
systemctl stop iscsid
systemctl disable iscsid
```

6. reboot


At the controller node
1.  edit the globals.yml file

enable cinder and cinder_backend_lvm

2. pull cinder images
'''
kolla-ansible pull
'''

3. Deploy
