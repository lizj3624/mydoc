- [Pod：运行于kubernetes中的容器](#pod运行于kubernetes中的容器)
  - [介绍POD](#介绍pod)
  - [创建POD](#创建pod)
  - [标签](#标签)
  - [命名空间](#命名空间)
- [副本机制](#副本机制)
- [服务service](#服务service)
# Pod：运行于kubernetes中的容器
## 介绍POD
Pod是一组并置的容器，是kubernetes调度和管理的最小单位
1. 一个Pod的所有容器都运行在同一个节点上，绝不运行跨节点。
2. 同一个Pod的容器都在相同的network和UTS命名空间下，他们共享相同的主机名和网络接口，也在相同的IPC命令空间下。由于同一Pod下共享网络接口，就拥有相同的IP和端口空间，因此不同进程不能监听相同端口。
3. kubernetes集群中的所有Pod都在同一个共享网络下，POD中容器能够像在无NAT的平坦网络中一样相互通信，就像局域网上的计算机一样。
4. 一个Pod可以放在相关联的服务容器，POD是逻辑主机，其行为与非容器世界中的物理主机非常相似。
5. 通过YAML和JSON创建Pod
   * metadata 包括名称、命名空间、标签和关于该容器 的其他信息 
   * spec 包含 pod 内容的实际说明 ， 例如 pod 的容器、卷和其他数据 
   * status包含运行中的pod的当前信息，例如pod所处的条件、 每个容器的描述和状态，以及内部 IP 和其他基本信息 。

```shell
# 根据YAML创建Pod资源
kubectl create -f kubia-manual.yaml
kubectl apply -f kubia-manual.yaml

# 获取Pod信息
kubectl get po kubia-manual -o yaml

# 查看所有Pod
kubectl get pods

# 查看日志
kubectl logs kubia-manual
```

6. 使用标签组织Pod
```shell
kubectl get po --show-labels

kubectl get po -L creation_method,env
```

7. 注解

8. 命名空间

9. 停止和移除
```shell
# 按名称删除 kubia-gpu pod
kubectl delete po kubia-gpu
```
## 创建POD
1. POD定义YAML文件主要组成
* apiVersion：kubernotes API版本
* kind：YAML描述的资源类型
* metadata：名称、命名空间、标签和关于容器的信息
* spec：POD内容的实际说明，POD的容器、卷和其他数据
* status：运行中POD的当前信息
  
```shell
# 通过yaml或json文件创建POD
kubectl create -f kubia-manual.yaml

# 查看POD
kubectl get pods

# POD日志
kubectl logs kubia-manual
```

## 标签
1. 创建和修改标签
```yaml
apiVersion: v1
kind: POD
metadata:
  name: kubia-manual-v2
  lables:
    creation_method: manual
    env: prod
spec:
  containers:
  - image: luksa/kubia
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
```

```shell
kubectl create -f kubia-manual-with-labels.yaml

# 查看pod的label信息
kubectl get po --show-labels

# 根据标签选择POD
kubectl get po -L creation_method,env

# 修改现有标签
kubectl label po kubia-manual-v2 env=debug --overwrite

kubectl get po -1 creation_method=manual

# 可以使用 !=, in, notint
```

## 命名空间
```shell
# 获取namespace
kubectl get ns

# 获取指定命名空间下的POD， --namespace可以用-n代替
kubectl get po --namespace kube-system
```

* 创建一个命名空间的YAML
```yaml
apiVersion: v1
kind: Namespace
metadata: 
  name: custom-namespace
```

```shell
# 通过YAML文件创建
kubectl create - f custom-namespace .yaml

# 通过命令创建
kubectl create namespace custom-namespace
```

3. 停止和移除POD
```shell
# 删除指定POD
kubectl delete po kubia-gpu

# 根据标签删除POD
kubectl delete po -l creation_method=manual

# 删除命名空间的POD
kubectl delete ns custom-namespace

# 删除当前命名空间下所有POD，保留ns
kubectl delete po --all
```

# 副本机制
```shell
# ReplicaSet
```

# 服务service
