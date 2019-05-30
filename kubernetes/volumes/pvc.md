### pvc存储卷
pvc的使用逻辑比较复杂。当创建pod需要使用pvc存储卷时，需要通过pvc说明使用多大的存储空间，而不需要知道后面实际的存储过程。
pod中使用的volumes必须与当前名称空间的pvc建立绑定关系，而pvc必须与某个实际的pv（pv是某个存储物理设备上的实际存储空间）建立绑定关系。
pvc与pv都是kubernetes集群中的资源类型。
pvc需要与某个pv绑定起来，就完成了pvc中的数据存放到绑定的pv上。
pvc需要绑定哪个pv，决定于创建pod时，需要使用多大的存储空间。通过pvc定义的请求大小，在pv中选择一个最合适的pv，建立绑定关系。

创建pod时，使用pvc存储卷，使用的字段说明：

      [root@docker1:~ ]# kubectl  explain pods.spec.volumes.persistentVolumeClaim
      KIND:     Pod
      VERSION:  v1

      RESOURCE: persistentVolumeClaim <Object>

      DESCRIPTION:
           PersistentVolumeClaimVolumeSource represents a reference to a
           PersistentVolumeClaim in the same namespace. More info:
           https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims

           PersistentVolumeClaimVolumeSource references the user's PVC in the same
           namespace. This volume finds the bound PV and mounts that volume for the
           pod. A PersistentVolumeClaimVolumeSource is, essentially, a wrapper around
           another type of volume that is owned by someone else (the system).

      FIELDS:
         claimName	<string> -required-    #pod使用claimName字段与pvc建立关系。
           ClaimName is the name of a PersistentVolumeClaim in the same namespace as
           the pod using this volume. More info:
           https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims

         readOnly	<boolean>
           Will force the ReadOnly setting in VolumeMounts. Default false.
   **claimName：表示pod使用此字段定义的名称与pvc建立关联关系。**

#### 什么是pvc？
pvc也是kubernetes中的一种资源类型。

创建pvc类型的资源时，使用的字段与其他类型资源一样，可以使用命令# kubectl  explain pvc查看。

spec字段常用子字段有：

      accessModes：定义pvc访问模式，是否支持多人挂载使用，是否支持多人读写等权限设定
      resources：定义资源限制，表示最少使用多少资源。比如 resources 5G，则pvc在绑定pv时，会选择5G以上的pv绑定。
      selector： 定义通过标签来选择pv，镜像绑定。
      volumeName： 定义卷名称，表示精确绑定到指定的pv上。
            
创建pod之前，需要先创建pvc，pvc表示在kubernetes中选择一个符合条件的pv来绑定使用。

绑定后，表示这个pvc与pv建立一一绑定关系。某pv被pvc绑定后，其状态为binding，那么其他pvc就不能在使用此pv。

pod与pvc存储卷是多对一绑定关系，可被多个pods使用。（具体能不能还是要看accessModes的设置）

#### 注意： pv是kubernetes集群中的资源，不属于某pod的名称空间。  pvc属于某个pod的名称空间。

pod-pvc-pv之间的关系：
      https://github.com/cxhzcxhz/notes/blob/master/kubernetes/volumes/images/pod-pvc-pv.png

pod与pvc关系：
      https://github.com/cxhzcxhz/notes/blob/master/kubernetes/volumes/images/pod%E4%B8%8Epvc%E5%A4%9A%E5%AF%B9%E4%B8%80%E7%BB%91%E5%AE%9A%E5%85%B3%E7%B3%BB.png


### 使用pvc存储卷，需要在kubernetes中提前定义好pv资源。
#### 制作pvc存储卷示例：
##### 第一步：定义nfs格式的pv示例：
前提准备，这里使用nfs server将共享的几个目录定义为pv。准备工作如下：

      [root@docker1:/data/volumes ]# mkdir -p {v1,v2,v3,v4,v5}
      [root@docker1:/data/volumes ]# cat /etc/exports
      /data/volumes/v1 192.168.12.0/24(rw,no_root_squash)
      /data/volumes/v2 192.168.12.0/24(rw,no_root_squash)
      /data/volumes/v3 192.168.12.0/24(rw,no_root_squash)
      /data/volumes/v4 192.168.12.0/24(rw,no_root_squash)
      /data/volumes/v5 192.168.12.0/24(rw,no_root_squash)
      [root@docker1:/data/volumes ]# systemctl  restart nfs
      [root@docker1:/data/volumes ]# showmount  -e docker1
      Export list for docker1:
      /data/volumes/v5 192.168.12.0/24
      /data/volumes/v4 192.168.12.0/24
      /data/volumes/v3 192.168.12.0/24
      /data/volumes/v2 192.168.12.0/24
      /data/volumes/v1 192.168.12.0/24
然后其他node节点挂载共享目录。

创建pv配置文件，在kubernetes集权中创建pv资源，示例：

      [root@docker1:~/mainfests/volumes ]# cat pv-demo.yaml 
      apiVersion: v1
      kind: PersistentVolume
      metadata:
        name: pv001
        labels:
          name: pv001
      spec:
        nfs:
          path: /data/volumes/v1
          server: 192.168.12.155
        accessModes: ["ReadWriteMany","ReadWriteOnce"]
        capacity:
          storage: 2Gi
      ---

      apiVersion: v1
      kind: PersistentVolume
      metadata:
        name: pv002
        labels:
          name: pv002
      spec:
        nfs:
          path: /data/volumes/v2
          server: 192.168.12.155
        accessModes: ["ReadWriteOnce"]
        capacity:
          storage: 5Gi
      ---
      apiVersion: v1
      kind: PersistentVolume
      metadata:
        name: pv003
        labels:
          name: pv003
      spec:
        nfs:
          path: /data/volumes/v3
          server: 192.168.12.155
        accessModes: ["ReadWriteMany","ReadWriteOnce"]
        capacity:
          storage: 20Gi
      ---
      apiVersion: v1
      kind: PersistentVolume
      metadata:
        name: pv004
        labels:
          name: pv004
      spec:
        nfs:
          path: /data/volumes/v4
          server: 192.168.12.155
        accessModes: ["ReadWriteMany","ReadWriteOnce"]
        capacity:
          storage: 10Gi
      ---
      apiVersion: v1
      kind: PersistentVolume
      metadata:
        name: pv005
        labels:
          name: pv005
      spec:
        nfs:
          path: /data/volumes/v5
          server: 192.168.12.155
        accessModes: ["ReadWriteMany","ReadWriteOnce"]
        capacity:
          storage: 10Gi
      ---

创建并查看的pv，命令：
      
      [root@docker1:~/mainfests/volumes ]# kubectl apply -f pv-demo.yaml 
      persistentvolume/pv001 created
      persistentvolume/pv002 created
      persistentvolume/pv003 created
      persistentvolume/pv004 created
      persistentvolume/pv005 created
      
      [root@docker1:~ ]# kubectl  get pv
      NAME    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
      pv001   2Gi        RWO,RWX        Retain           Available                                   3s
      pv002   5Gi        RWO            Retain           Available                                   3s
      pv003   20Gi       RWO,RWX        Retain           Available                                   3s
      pv004   10Gi       RWO,RWX        Retain           Available                                   3s
      pv005   10Gi       RWO,RWX        Retain           Available                                   3s

到此已经完成了创建pv过程。

知识点：

      Pv的访问模式：https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
            访问模式是：
                  ReadWriteOnce - 卷可以由单个节点以读写方式挂载
                  ReadOnlyMany - 卷可以由许多节点以只读方式挂载
                  ReadWriteMany - 卷可以由许多节点以读写方式挂载
      
##### 第二步：定义pvc配置文件，示例：





