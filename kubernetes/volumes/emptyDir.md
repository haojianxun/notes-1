### emptyDir示例
#### emptyDir有2个字段，medium和medium
#### emptyDir: {}  表示使用默认值，而不是不定义其子字段。
#### emptyDir需要在pods的volumes中定义，在containers中通过volumesMounts字段挂载使用。

**创建emptyDir配置文件**

    [root@docker1:~/mainfests/volumes ]# cat pod-empty-demo.yaml
    apiVersion: v1
    kind: Pod
    metadata:
       name: pod-demo-emptydir
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
         - name: html
           mountPath: /usr/share/nginx/html/
       - name: busybox
         image: busybox:latest
         command:
         - "/bin/sh"
         - "-c"
         - "while true; do echo  $(date) >> /data/index.html; sleep 2 ; done"
         volumeMounts:
         - name: html
           mountPath: /data/
       - name: httpd 
         image: busybox:latest
         command: ["/bin/httpd","-f","-h","/tmp","-p","81"]
         volumeMounts:
         - name: html
           mountPath: /data/web/html/
       volumes:
       - name: html
         emptyDir: {}
##### volumeMounts：通过volumesMounts字段进行存储卷挂载
##### name：表示要挂在的存储卷名称。（过载哪个存储卷）
##### mountPath：表示将存储卷挂载到容器的哪个路径
##### volumes定义存储卷，此为emptyDir类型的存储卷（先【在pods级别】定义，后【在containers级别】挂载）

这个pod中包含了3个container，这里有个提醒，多个容器时，注意容器监听的端口，不要冲突。
这里myapp容器镜像监听80端口，httpd容器镜像监听81端口，这样启动正确。

**创建pods实例**

    [root@docker1:~/mainfests/volumes ]# kubectl  apply -f pod-empty-demo.yaml 
    pod/pod-demo-emptydir created

**查看创建的实例**

    [root@docker1:~ ]# kubectl  get pods  -o wide
    NAME                             READY   STATUS    RESTARTS   AGE     IP            NODE      NOMINATED NODE   READINESS GATES
    pod-demo                         3/3     Running   0          89m     10.244.1.40   docker2   <none>           <none>
    pod-demo-emptydir                3/3     Running   0          6s      10.244.2.44   docker3   <none>           <none>
    pod-demo-gitrepo                 2/2     Running   0          50m     10.244.1.41   docker2   <none>           <none>
    pod-vol-emptydir                 1/1     Running   1          21h     10.244.2.35   docker3   <none>           <none>
    tomcat-deploy-8475677b49-fc722   1/1     Running   0          4h50m   10.244.2.39   docker3   <none>           <none>
    tomcat-deploy-8475677b49-nrd5m   1/1     Running   0          4h50m   10.244.2.38   docker3   <none>           <none>
    tomcat-deploy-8475677b49-tbx42   1/1     Running   0          174m    10.244.2.43   docker3   <none>           <none>

