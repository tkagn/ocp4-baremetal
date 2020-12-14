# Uninstall OCS
Openshift Container Storage (OCS) is best uninstalled from the command-line (cli) to ensure all of its components are properly removed. The web console (UI) does not remove all the components and may cause some of the dependent component to be properly removed.

## Delete the StorageCluster object

Delete the StorageCluster object and wait for the removal of the associated resources.

```bash
oc delete -n openshift-storage storagecluster --all --wait=true

```

## Remove CustomResourceDefinitions

```bash
oc delete crd backingstores.noobaa.io bucketclasses.noobaa.io cephblockpools.ceph.rook.io cephclusters.ceph.rook.io cephfilesystems.ceph.rook.io cephnfses.ceph.rook.io cephobjectstores.ceph.rook.io cephobjectstoreusers.ceph.rook.io noobaas.noobaa.io ocsinitializations.ocs.openshift.io  storageclusterinitializations.ocs.openshift.io objectbuckets.objectbucket.io objectbucketclaims.objectbucket.io storageclusters.ocs.openshift.io  --wait=true --timeout=5m
```

## Delete CSVs in openshift-storage namespace
```bash
for i in $(oc get -n openshift-storage csv -o jsonpath='{ .items[*].metadata.name  }'); do oc delete csv ${i} -n openshift-storage; done
```

## Delete remaining pods, deamonsets, replicasets, deployment.apps, and services

```bash
oc delete pods,jobs,ds,rs,statefulset,hpa,deployment.apps,jobs,service,route --all -n openshift-storage
```

## Delete CSVs in openshift-storage namespace
```bash
for i in $(oc get -n openshift-storage csv -o jsonpath='{ .items[*].metadata.name  }'); do oc delete csv ${i} -n openshift-storage; done
```

## Verify no resources in openshift storage namespace
```bash
oc get all -n openshift-storage
```

## Delete the namespace and wait till the deletion is complete.
```bash
oc delete project openshift-storage --wait=true --timeout=5m
oc delete namespace openshift-storage
```

## If using a cloud provider (AWS, Google, Azure, etc) scale the machinesets to zero to remove storage nodes
```bash
oc get machinesets.machine.openshift.io -n openshift-machine-api
oc scale --replicas=0 machineset <machinesetname> -n openshift-machine-api
```

## Optional: If re-using nodes, clean up the storage operator artifacts on each node.
```bash
for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /var/lib/rook; done
for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /var/lib/kubelet/plugins; done
for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /var/lib/kubelet/plugins_registry; done

for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /etc/lvm/backup/; done
for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /etc/lvm/archive/; done


for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /var/lib/ceph; done
for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /etc/ceph; done

for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /var/log/ceph; done
for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /dev/termination-log; done
for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /tmp/operator-sdk-ready; done
for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /etc/ceph-csi-config/; done
for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /tmp/csi/keys; done
for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /etc/rook/; done

for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host mkdir -p /var/lib/kubelet/plugins; done

for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host mkdir -p /var/lib/kubelet/plugins_registry; done
```

## Delete the storage classes with an openshift-storage provisioner listed in step 1.
```bash
oc delete storageclass ocs-storagecluster-ceph-rbd ocs-storagecluster-cephfs openshift-storage.noobaa.io --wait=true --timeout=5m
```

## Delete the local volume 
```bash
LV=local-block
SC=localblock
```

## List and note the devices to be cleaned up later.
```bash
oc get localvolume -n local-storage $LV -o jsonpath='{ .spec.storageClassDevices[*].devicePaths[*] }'
```


## Delete the local volume resource.
```bash
oc delete localvolume -n local-storage --wait=true $LV
```

## Delete the remaining PVs and StorageClasses if they exist.
```bash
oc delete pv -l storage.openshift.com/local-volume-owner-name=${LV} --wait --timeout=5m
oc delete storageclass $SC --wait --timeout=5m
```

## Delete local-storage 
```bash
oc delete pods,services,deployment.apps,replicaset.apps --all -n local-storage
oc delete project local-storage --wait=true --timeout=5m
oc delete namespace local-storage
```

## Clean up the artifacts from the storage nodes for that resource.
```bash
[[ ! -z $SC ]] && for i in $(oc get node -l cluster.ocs.openshift.io/openshift-storage= -o jsonpath='{ .items[*].metadata.name }'); do oc debug node/${i} -- chroot /host rm -rfv /mnt/local-storage/${SC}/; done
```

## Wipe the disks for each of the local volumes listed in step 4 so that they can be reused on each storage node.
```bash
oc debug node/<nodename>
chroot /host
DISKS="/dev/disk/by-id/wwn-0x614187705a2c050026a67c4607b3d1cd /dev/disk/by-id/wwn-0x614187705a2c020026a67c1c0546807d /dev/disk/by-id/wwn-0x61866da0577e830026a67beb066415ea" 
for disk in $DISKS; do sgdisk --zap-all $disk;done

rm -rfv /etc/lvm/backup/ /etc/lvm/archive/ /dev/loop0 /dev/loop1
rm -rfv /etc/lvm/archive/

rm -rfv /dev/loop0
rm -rfv /dev/loop1

dmsetup remove 
```

## Unlabel the storage nodes.
```bash
oc label nodes  --all cluster.ocs.openshift.io/openshift-storage-
oc label nodes  --all topology.rook.io/rack-

oc get nodes -l cluster.ocs.openshift.io/openshift-storage=
```
