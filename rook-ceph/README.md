## rbd kernel module

```sh 
modprobe rbd
```

## USB disk SCSI 로 인식시키기

https://github.com/rook/rook/issues/14699

`/etc/udev/rules.d/99-usd-to-scsi.rules`:
```
ACTION=="add", ENV{ID_TYPE}=="disk", ENV{ID_BUS}=="usb", ENV{ID_BUS}="scsi"
ACTION=="change", ENV{ID_TYPE}=="disk", ENV{ID_BUS}=="usb", ENV{ID_BUS}="scsi"
ACTION=="online", ENV{ID_TYPE}=="disk", ENV{ID_BUS}=="usb", ENV{ID_BUS}="scsi"
```

## filesystem 삭제

(unmount 된 상태에서)
```sh
wipefs --all /dev/sdX
```

## rook-ceph

- https://github.com/rook/rook/blob/master/deploy/examples/crds.yaml
- https://github.com/rook/rook/blob/master/deploy/examples/common.yaml
- https://github.com/rook/rook/blob/master/deploy/examples/operator.yaml
- https://github.com/rook/rook/blob/master/deploy/examples/cluster.yaml
    - allowMultiplePerNode: true
- https://github.com/rook/rook/blob/master/deploy/examples/toolbox.yaml

```
bash-5.1$ ceph osd status
ID  HOST    USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE
 0  tmcha   426M  13.9G      0        0       0        0   exists,up
```

## StorageClass

https://rook.io/docs/rook/latest/CRDs/Block-Storage/ceph-block-pool-crd/

node 1대, osd 1 개라서 살짝 수정해준다.
```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: osd
  replicated:
    size: 1
    requireSafeReplicaSize: false
```

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-ceph-block
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
  clusterID: rook-ceph 
  pool: replicapool
  imageFormat: "2"
  imageFeatures: layering
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph 
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
  csi.storage.k8s.io/fstype: ext4
allowVolumeExpansion: true
reclaimPolicy: Delete
```
