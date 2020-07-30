---
title: K8S笔记Pod相关(四)
date: 2020-07-30 09:28:19
tags: [K8S笔记]
---

# K8S笔记Pod相关(四)
## Pod的介绍
Pod vs  应用  
每个Pod都是应用的一个实例，有专用的IP  
Pod vs  容器  
```
一个Pod可以有多个容器，彼此间共享网络和存储资源，
每个Pod 中有一个Pause容器保存所有的容器状态，
通过管理pause容器，达到管理pod中所有容器的效果
```
Pod vs  节点  
```
同一个Pod中的容器总会被调度到相同Node节点，不同节点间Pod的通信基于虚拟二层网络技术实现  
```
Pod vs Pod  
```
普通的Pod和静态Pod  
```
<!--more-->

## Pod的介绍
下面是yaml文件定义的Pod的完整内容
```
apiVersion : v1              //版本
kind: Pod                    //类型，pod
metadata:                    //元数据
  name: string               //元数据，pod的名字
  namespace: string          //元数据，pod的命名空间
  labels:                    //元数据，标签列表
    - name: string           //元数据，标签的名字
  annotations:               //元数据,自定义注解列表
    - name: string           //元数据,自定义注解名字
spec:                        //pod中容器的详细定义
  containers :               //pod中的容器列表，可以有多个容器
  - name: string             //容器的名称
    image: string            //容器中的镜像
    imagesPullPolicy: [Always|Never|IfNotPresent]
    //获取镜像的策略，默认值为Always，每次都尝试重新下载镜像
    command: [string] 
    //容器的启动命令列表（不配置的话使用镜像内部的命令）
    args: [string]              //启动参数列表
    workingDir: string          //容器的工作目录
    volumeMounts:               //挂载到到容器内部的存储卷设置
    - name: string
        mountPath: string         //存储卷在容器内部Mount的绝对路径
        readOnly: boolean         //默认值为读写
    ports:                        //容器需要暴露的端口号列表
    - name: string
      containerPort: int          //容器要暴露的端口
      hostPort: int               //容器所在主机监听的端口（容器暴露端口映射到宿主机的端口，设置hostPort时同一
台宿主机将不能再启动该容器的第2份副本）
      protocol: string           //TCP和UDP，默认值为TCP
    env:                        //容器运行前要设置的环境列表
    - name: string
      value: string
    resources:
      limits:                    //资源限制，容器的最大可用资源数量
        cpu: Srting
        memory: string
      requeste:                  //资源限制，容器启动的初始可用资源数量
        cpu: string
        memory: string
    livenessProbe:               //pod内容器健康检查的设置        
      exec:
        command: [string]        //exec方式需要指定的命令或脚本
      httpGet:                   //通过httpget检查健康
        path: string
        port: number
        host: string
        scheme: Srtring
        httpHeaders:
        - name: Stirng
          value: string
      tcpSocket:                   //通过tcpSocket检查健康
        port: number
      initialDelaySeconds: 0       //首次检查时间
      timeoutSeconds: 0            //检查超时时间
      periodSeconds: 0             //检查间隔时间
      successThreshold: 0
      failureThreshold: 0
      securityContext:             //安全配置
        privileged: falae        
    restartPolicy: [Always|Never|OnFailure]
    //重启策略，默认值为Always
    nodeSelector: object //节点选择，表示将该Pod调度到包含这些label的Node上，以key:value格式指定
    imagePullSecrets:
    - name: string
    hostNetwork: false      //是否使用主机网络模式，弃用Docker网桥，默认否
  volumes:                      //在该pod上定义共享存储卷列表
    - name: string
      emptyDir: {}    //是一种与Pod同生命周期的存储卷，是一个临时目录，内容为空
      hostPath:       //Pod所在主机上的目录，将被用于容器中mount的目录
        path: string
      secret:              //类型为secret的存储卷
        secretName: string  
        item:
        - key: string
          path: string
      configMap:                //类型为configMap的存储卷
        name: string
        items:
        - key: string
          path: string    
```

## 基本用法
```
在kubernetes中对运行容器的要求为：容器的主程序需要一直在前台运行，而不是后台运行。
应用需要改造成前台运行的方式。如果我们创建的Docker镜像的启动命令是后台执行程序，
则在kubelet创建包含这个容器的pod之后运行完该命令，即认为Pod已经结束，将立刻销毁该Pod。
如果为该Pod定义了RC，则创建、销毁会陷入一个无限循环的过程中。

Pod可以由1个或多个容器组合而成。

```
#### 由一个容器组成的 Pod示例
```
# 一个容器组成的Pod
apiVersion: v1
kind: Pod
metadata:
  name: mytomcat
  labels:
    name: mytomcat
spec:
  containers:
  - name: mytomcat
    image: tomcat
    ports:
    - containerPort: 8000
```
#### 由两个为紧耦合的容器组成的 Pod示例
```
#两个紧密耦合的容器
apiVersion: v1
kind: Pod
metadata:
  name: myweb
  labels:
    name: tomcat-redis
spec:
  containers:
  - name: tomcat
    image: tomcat
    ports:
    - containerPort: 8080
  - name: redis
    image: redis
    ports:
    - containerPort: 6379
```

## Pod操作命令
#### 创建
```
kubectl create -f xxx.yaml
```
#### 查看
```
kubectl get pod <Pod_name>
kubectl get pod <Pod_name> -o wide
//查看详细信息
kubectl describe pod <Pod_name>
//查看所有
kubectl get pods
```
#### 删除
```
kubectl delete -f pod pod_name.yaml
kubectl delete pod --all/[pod_name]
```

## Pod生命周期
![Pod生命周期](/img/2020-07-25/16.png)

## Pod重启策略
![Pod重启策略](/img/2020-07-25/17.png)

## Pod状态转换
![Pod状态转换](/img/2020-07-25/18.png)

## 资源分配
```
每个Pod都可以对其能使用的服务器上的计算资源设置限额，Kubernetes中可以设置限额的计算资源有CPU与
Memory两种
，其中CPU的资源单位为CPU数量,是一个绝对值而非相对值。
Memory配额也是一个绝对值，它的单位是内存字节数。

Kubernetes里，
一个计算资源进行配额限定需要设定以下两个参数：
`Requests`  该资源最小申请数量，系统必须满足要求
`Limits`    该资源最大允许使用的量，不能突破，当容器试图使用超过这个量的资源时，可能会被Kubernetes Kill并重启

```
例子:
```
sepc
  containers:
  - name: db
    image: mysql
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
上述代码表明MySQL容器申请最少0.25个CPU以及64MiB内存，
在运行过程中容器所能使用的资源配额为0.5个
CPU以及128MiB内存。