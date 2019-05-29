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
