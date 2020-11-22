# 带有cdrom的虚拟机测试结果

```bash 

[root@node1 centos]# kubectl get vmi
NAME        AGE   PHASE     IP                  NODENAME
centos711   30m   Running   172.16.166.180/32   node1
[root@node1 centos]#
[root@node1 centos]#
[root@node1 centos]# ssh 172.16.166.180
The authenticity of host '172.16.166.180 (172.16.166.180)' can't be established.
ECDSA key fingerprint is SHA256:5LKHfVRd4Gd/Vtu6es+yyD6Fw0BHyzm5Jh6Fnw3CYSc.
ECDSA key fingerprint is MD5:9c:4f:98:6a:c8:11:6c:7d:3e:c4:a0:1e:50:71:2a:e3.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.166.180' (ECDSA) to the list of known hosts.
root@172.16.166.180's password:
[root@centos711 ~]#
[root@centos711 ~]#
[root@centos711 ~]# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sr0     11:0    1   2M  0 rom
vda    253:0    0  23G  0 disk
└─vda1 253:1    0   8G  0 part /
[root@centos711 ~]# mount /dev/sr0  /media/
mount: /dev/sr0 is write-protected, mounting read-only
[root@centos711 ~]# cd /media/
[root@centos711 media]# ls
hostname
[root@centos711 media]# cat hostname
cirros-dv-cdrom


```

创建kvm时加一个cdrom device，跟k8s中的pvc对应， 

```
[root@node1 centos]# cat pvc2.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: "centos711-init-disk"
  labels:
    app: containerized-data-importer
  annotations:
    cdi.kubevirt.io/storage.import.endpoint: "http://192.168.122.1:7001/kvm-image/cdrom.iso"
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Mi
  storageClassName: hostpath
```


其内容来源于一个iso文件， iso 里面可以预先放入一些信息。

```bash 

[root@node1 kvm-image]# mount -o loop cdrom.iso  /media/
mount: /dev/loop0 is write-protected, mounting read-only
[root@node1 kvm-image]# cd /media/
[root@node1 media]# ls
hostname
[root@node1 media]# cat hostname
cirros-dv-cdrom
```

# 完整的vm yaml文件

centos7 vm 挂有2个盘，一个是系统盘，一个是初始化数据盘
```
[root@node1 centos]# cat vm.yaml
apiVersion: kubevirt.io/v1alpha3
kind: VirtualMachine
metadata:
  creationTimestamp: 2018-07-04T15:03:08Z
  generation: 1
  labels:
    kubevirt.io/os: linux
  name: centos711
spec:
  running: true
  template:
    metadata:
      creationTimestamp: null
      labels:
        kubevirt.io/domain: centos711
    spec:
      domain:
        cpu:
          cores: 2
        devices:
          disks:
          - disk:
              bus: virtio
            name: disk0
          - cdrom:
              bus: sata
              readonly: true
            name: disk1
        machine:
          type: q35
        resources:
          requests:
            memory: 1024M
      volumes:
      - name: disk0
        persistentVolumeClaim:
          claimName: centos711-system-disk
      - name: disk1
        persistentVolumeClaim:
          claimName: centos711-init-disk
```

- 系统盘 

```
[root@node1 centos]# cat pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: "centos711-system-disk"
  labels:
    app: containerized-data-importer
  annotations:
    cdi.kubevirt.io/storage.import.endpoint: "http://192.168.122.1:7001/kvm-image/CentOS-7-2003-02.qcow2"
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 23Gi
  storageClassName: hostpath
```

- 初始化数据cdrom 

```
[root@node1 centos]# cat pvc2.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: "centos711-init-disk"
  labels:
    app: containerized-data-importer
  annotations:
    cdi.kubevirt.io/storage.import.endpoint: "http://192.168.122.1:7001/kvm-image/cdrom.iso"
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Mi
  storageClassName: hostpath
```




