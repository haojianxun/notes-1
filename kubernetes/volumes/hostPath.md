### hostPath存储卷
#### 简单来说，就是把pod所在宿主机之上的，脱离pod中容器名称空间之外的，宿主机的文件系统目录与pod建立关联关系。所以删除了pod，数据还在本地宿主机上，只要新创建的pod能调度到该宿主机上来，就能使用之前的数据。
> hostPath有2个子字段，path、type
>>  path：用来表示宿主机上的路径信息
>>  type：用来指定hostPath存储卷类型
>>  type支持的字段值有；
>>>   DirectoryOrCreate：如果path指定的目录路径不存在，则创建一个空目录，且权限为0755，与kubelet拥有相同的属主属组。
>>>   Directory：指定path路径必须存在，否则启动失败。
      FileOrCreate：如果path指定的文件路径不存在，则创建一个空文件，且权限为0644，与kubelet拥有相同的属组属主。
      File：path指定的文件必须存在，否则启动报错。
      Socket：path指定的unix socket路径必须存在。
      CharDevice：path指定的路径必须是个字符设备。
      BlockDevice：path指定的路径必须是个块设备。
