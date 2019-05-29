### nfs存储卷
常用的nfs存储卷字段说明：

    [root@docker1:~ ]# kubectl  explain pods.spec.volumes.nfs
    KIND:     Pod
    VERSION:  v1

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

下面实验中，使用master节点docker1作为nfs server，提供共享存储使用的共享目录，使用node节点docker2、docker3挂载nfs共享目录。

前提准备工作：

    docker1：

        [root@docker1:~ ]# yum install nfs-utils -y
        [root@docker1:~ ]# mkdir -p /data/volumes
        [root@docker1:~ ]# cat /etc/exports
        /data/volumes 192.168.12.0/24(rw,no_root_squash)
        [root@docker1:~ ]# cat /data/volumes/index.html  #创建个默认主页
        this is nfs server index

    docker2：

        [root@docker2 ~]# yum install nfs-utils -y
        [root@docker2 ~]# mkdir -p  /mnt/nfs
        [root@docker2 ~]# mount docker1:/data/volumes/ /mnt/nfs/
        [root@docker2 ~]# mount |grep "/mnt/nfs"
        docker1:/data/volumes on /mnt/nfs type nfs4 (rw,relatime,vers=4.1,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.12.174,local_lock=none,addr=192.168.12.155)

    docker3：

        [root@docker3 ~]# yum install nfs-utils -y
        [root@docker3 ~]# mkdir -p /mnt/nfs
        [root@docker3 ~]# mount docker1:/data/volumes/ /mnt/nfs/
        [root@docker3 ~]# mount | grep "/mnt/nfs"
        docker1:/data/volumes on /mnt/nfs type nfs4 (rw,relatime,vers=4.1,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.12.189,local_lock=none,addr=192.168.12.155)

#### 创建nfs存储卷示例：

    [root@docker1:~/mainfests/volumes ]# vim pod-nfs-vol.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
       name: pod-vol-nfs
       namespace: default
    spec:
       containers:
       - name: myapp
         image: ikubernetes/myapp:v1
         volumeMounts:
         - name: html
           mountPath: /usr/share/nginx/html/
       volumes:
       - name: html
         nfs:
            path: /data/volumes
            server: 192.168.12.155

    [root@docker1:~/mainfests/volumes ]# kubectl  apply -f pod-nfs-vol.yaml 
    pod/pod-vol-nfs created
    [root@docker1:~/mainfests/volumes ]# kubectl  get pods -o wide
    NAME                             READY   STATUS    RESTARTS   AGE    IP            NODE      NOMINATED NODE   READINESS GATES
    pod-vol-hostpath                 1/1     Running   1          3h9m   10.244.1.47   docker2   <none>           <none>
    pod-vol-nfs                      1/1     Running   0          18s    10.244.1.48   docker2   <none>           <none>
    tomcat-deploy-8475677b49-fc722   1/1     Running   1          27h    10.244.2.45   docker3   <none>           <none>
    tomcat-deploy-8475677b49-nrd5m   1/1     Running   1          27h    10.244.2.46   docker3   <none>           <none>
    tomcat-deploy-8475677b49-tbx42   1/1     Running   1          25h    10.244.2.47   docker3   <none>           <none>
    [root@docker1:~/mainfests/volumes ]# curl 10.244.1.48
    this is nfs server index




