### 存储卷使用gitRepo示例，但是官方提醒要弃用了。

**创建gitrepo示例:**

    [root@docker1:~/mainfests/volumes ]# cat  pod-gitrepo-demo.yaml
    apiVersion: v1
    kind: Pod
    metadata:
       name: pod-demo-gitrepo
       namespace: default
       labels:
         app: myapp
         tier: frontend
    spec:
       containers:
       - name: myapp
         image: ikubernetes/myapp:v1
         ports:
         - name: http
           containerPort: 80
         volumeMounts:
         - name: gitrepo 
           mountPath: /usr/share/nginx/html/
       - name: busybox
         image: busybox:latest
         command:
         - "/bin/sh"
         - "-c"
         - "while true; do echo  $(date) >> /data/index.html; sleep 2 ; done"
         volumeMounts:
         - name: gitrepo
           mountPath: /data/
       volumes:
       - name: gitrepo
         gitRepo:
            repository: "https://github.com/cxhzcxhz/flow-web.git"
            revision: "e0d1c45"
            directory: "."

**查看创建的pods，命令：**

    [root@docker1:~ ]# kubectl  get pods  -o wide
    NAME                             READY   STATUS    RESTARTS   AGE     IP            NODE      NOMINATED NODE   READINESS GATES
    pod-demo                         3/3     Running   0          51m     10.244.1.40   docker2   <none>           <none>
    <u>pod-demo-gitrepo                 2/2     Running   0          11m     10.244.1.41   docker2   <none>           <none></u>
    pod-vol-emptydir                 1/1     Running   1          21h     10.244.2.35   docker3   <none>           <none>
    tomcat-deploy-8475677b49-fc722   1/1     Running   0          4h12m   10.244.2.39   docker3   <none>           <none>
    tomcat-deploy-8475677b49-nrd5m   1/1     Running   0          4h12m   10.244.2.38   docker3   <none>           <none>
    tomcat-deploy-8475677b49-tbx42   1/1     Running   0          135m    10.244.2.43   docker3   <none>           <none>

**进入pod-demo-gitrepo的myapp容器，查看挂在信息：**

    [root@docker1:~ ]# kubectl  exec -it pod-demo-gitrepo -c myapp  --  /bin/sh
    / # ls
    bin    dev    etc    home   lib    media  mnt    proc   root   run    sbin   srv    sys    tmp    usr    var
    / # cd /usr/share/
    /usr/share # ls
    GeoIP  man    misc   nginx
    /usr/share # cd nginx/html/
    /usr/share/nginx/html # ls
    Dockerfile  README.md   haha        index.html
    
    
  与我github仓库内容一致。

