For deploying swift, you need to have a running cloud and prepare the storage node for swift.
add 3 new disk to yr storage node /dev/sdb /dev/sdc /dev/sdd

1. Create a partition on each disk
```
$ sudo parted /dev/sdc -s -- mklabel gpt mkpart KOLLA_SWIFT_DATA 1 -1
```
do it on all of them make appropriate changes.

2. Create a file system
```
$ sudo mkfs.xfs -f -L d0 /dev/sdb1
$ sudo mkfs.xfs -f -L d1 /dev/sdc1
$ sudo mkfs.xfs -f -L d2 /dev/sdd1
```
3. Create a script to initialize swift ring on the controller node
```
nano init-swift
```
4. create a script to build swift rings(refer to init-swift)

5. Add execution permission to the script
```
$ sudo chmod 755 init-swift
```
6.  Run the script on the controller node

7. Before running deploy you need to enable swift on your globals file and pull the images. Then run the deployment
