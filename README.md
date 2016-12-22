# kube-addons

---
##组件
> kubernetes除了必备的dns和网络组件外,官方推出大量的`cluster-monitoring`,`dashboard`,`fluentd-elasticsearch`,`node-problem-detector`,`registry`

`官方提供的大部分组件,都以NodePort暴露服务,并且只允许在master节点上`

###heapster
> k8s的监控组件,自动伸缩与及Dashboard 都依赖与它.

```
cd /data
git clone https://github.com/kubernetes/heapster.git
kubectl apply -f /data/heapster/deploy/kube-config/influxdb/
```
> influxdb 默认没有配置数据持久化, 可以结合各自的数据持久化方案进行部署,以便保留监控历史

`安装完heapster组件后,可以便捷使用`kubectl top node`,`kubectl top pod` 列出高负载的资源`
```
kubectl top node -n kube-system  
NAME           CPU(cores)   CPU%      MEMORY(bytes)   MEMORY%   
10-8-44-35     162m         4%        1045Mi          13%       
10-8-50-182    77m          1%        1132Mi          14%       
10-8-113-246   156m         3%        674Mi           8%        
10-8-36-34     166m         4%        904Mi           11%
```

```
kubectl top pod -n kube-system   
NAME                                   CPU(cores)   MEMORY(bytes)   
kube-dns-2924299975-363jm              1m           32Mi            
kube-apiserver-10-8-113-246            6m           84Mi            
dummy-2088944543-05z36                 0m           0Mi             
kube-proxy-mcb3t                       18m          18Mi            
calico-policy-controller-shv0f         4m           16Mi            
canal-node-rc2t6                       2m           57Mi        
```

###dashboard
> 由kubernetes的UI演变而来, 目前已经集成监控展示与日常的创建与删除操作, 另一方面,可以通过ui界面来学习kubernetes的使用,认识常用的配置类型
> https://github.com/kubernetes/dashboard#kubernetes-dashboard
```
kubectl get pods --all-namespaces | grep dashboard
```

```
kubectl get po,svc --all-namespaces | grep dashboard    
kube-system   po/kubernetes-dashboard-3095304083-ltq8k   1/1       Running   0          53s
kube-system   svc/kubernetes-dashboard   10.105.98.204    <nodes>       80:32624/TCP    53s
```

>  通过10.105.98.204 进行访问

###node-problem-detector
> node经常会遇到以下问题:
> * 硬件问题:  cpu 内存 磁盘
> * 内核问题: 内核死锁, 文件系统损坏
> * 容器问题: 守护进程无响应

> kubernetes集群管理对node的健康状态是无法感知的, pod依旧会调度到有问题的node上,  通过DaemonSet部署node-problem-detector, 向apiserver上报node的状态信息,使node的健康状态对上游管理可见,pod不会再调度到有异常的node上.

> https://github.com/kubernetes/node-problem-detector

```
wget https://raw.githubusercontent.com/kubernetes/node-problem-detector/master/config/kernel-monitor.json
mkdir config
mv kernel-monitor.json config/
kubectl create configmap node-problem-detector-config --from-file=config/ -n kube-system
wget https://raw.githubusercontent.com/kubernetes/node-problem-detector/master/node-problem-detector.yaml
kubectl apply -f node-problem-detector.yaml -n kube-system
```

###DNS Horizontal Autoscaler
> 根据apiserver获取集群的状态信息, 实现dns服务的水平扩展, 有助于提高dns的服务能力. 

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/dns-horizontal-autoscaler/dns-horizontal-autoscaler.yaml
```
##fluentd-elasticsearch
> fluentd-elasticsearch 是kubernetes 官方提供的容器日志收集方案, 个人也认为,这是目前最好的面向kubernetes日志方案.官网的版本太旧,我制作了一个最新版本的, 只收集容器的日志.

```
git clone https://github.com/caiwenhao/kube-addons.git
cd kube-addons/fluentd-elasticsearch
kubectl apply -f es-deploy.yaml -f kibana-deploy.yaml -f es-deploy.yaml 
kubectl label node 10-8-50-182 alpha.kubernetes.io/fluentd-ds-ready=true
```
> 1. 通过elasticsearch-cloud-kubernetes,实现es在k8s集群上的部署. 
> 2.  fluentd 的插件`fluent-plugin-kubernetes_metadata_filter` 通过apiserver,当pod创建的时候,建立日志目录映射关系, 并解析kubernetes日志格式.

**进一步优化**
> 把

```

```