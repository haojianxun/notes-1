### nfs存储卷
常用的nfs存储卷字段说明：

  [root@docker1:~ ]# kubectl  explain pods.spec.volumes.nfs
  KIND:     Pod
  VERSION:  v1

  RESOURCE: nfs <Object>

  DESCRIPTION:
  
       NFS represents an NFS mount on the host that shares a pod's lifetime More
       info: https://kubernetes.io/docs/concepts/storage/volumes#nfs

       Represents an NFS mount that lasts the lifetime of a pod. NFS volumes do
       not support ownership management or SELinux relabeling.

  FIELDS:
  
     path	<string> -required-
       Path that is exported by the NFS server. More info:
       https://kubernetes.io/docs/concepts/storage/volumes#nfs

     readOnly	<boolean>
       ReadOnly here will force the NFS export to be mounted with read-only
       permissions. Defaults to false. More info:
       https://kubernetes.io/docs/concepts/storage/volumes#nfs

     server	<string> -required-
       Server is the hostname or IP address of the NFS server. More info:
       https://kubernetes.io/docs/concepts/storage/volumes#nfs
