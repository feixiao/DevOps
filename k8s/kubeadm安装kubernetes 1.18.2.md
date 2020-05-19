## kubeadm安装kubernetes 1.18.2

#### 准备基础环境
+ 修改hostname
  ```
  hostnamectl set-hostname master.k8s.com
  # 查看修改结果
  hostnamectl status
  # 设置 hostname 解析
  echo "127.0.0.1   $(hostname)" >> /etc/hosts
  ```
+ 校对时间
	```shell
	yum -y install ntp
	systemctl start ntpd
	systemctl enable ntpd
	```

#### docker、kubeadm、kubelet、kubectl
```shell
# 在 master 节点和 worker 节点都要执行
# 最后一个参数 1.18.2 用于指定 kubenetes 版本，支持所有 1.18.x 版本的安装
# 阿里云 docker hub 镜像
export REGISTRY_MIRROR=https://registry.cn-hangzhou.aliyuncs.com
curl -sSL https://kuboard.cn/install-script/v1.18.x/install_kubelet.sh | sh -s 1.18.2
```

可能出现的问题：
```
Failed to start docker.service: Unit not found.

去路径/etc/systemd/system/docker.service.requires下面删除无关的服务，我之前安装过flanneld.service，删除既可以运行docker

```

#### mastr节点
```shell
# 只在 master 节点执行
# 替换 x.x.x.x 为 master 节点实际 IP（请使用内网 IP）
# export 命令只在当前 shell 会话中有效，开启新的 shell 窗口后，如果要继续安装过程，请重新执行此处的 export 命令
export MASTER_IP=172.17.20.104

# 替换 apiserver.demo 为 您想要的 dnsName
export APISERVER_NAME=apiserver.demo

# Kubernetes 容器组所在的网段，该网段安装完成后，由 kubernetes 创建，事先并不存在于您的物理网络中
export POD_SUBNET=10.100.0.1/16
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts
curl -sSL https://kuboard.cn/install-script/v1.18.x/init_master.sh | sh -s 1.18.2
```

+ 启动失败恢复

  ```shell
  kubeadm reset
  rm -rf /var/lib/etcd/
  rm -rf /root/.kube/*
  ```

#### node节点

```shell
# 只在 worker 节点执行
# 替换 x.x.x.x 为 master 节点的内网 IP
export MASTER_IP=172.17.20.104
# 替换 apiserver.demo 为初始化 master 节点时所使用的 APISERVER_NAME
export APISERVER_NAME=apiserver.demo
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts

# master 节点上 kubeadm token create --print-join-command 获取下面的内容
kubeadm join apiserver.demo:6443 --token mpfjma.4vjjg8flqihor4vt     --discovery-token-ca-cert-hash sha256:6f7a8e40a810323672de5eee6f4d19aa2dbdb38411845a1bf5dd63485c43d303
```


### Dashboard
```shell
kubectl apply -f ./recommended.yaml
kubectl proxy

# 浏览器访问相关地址
```shell
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/overview?namespace=default

需要使用下面生成的token
```

##### 生成token
```shell
kubectl create -f ./account.yaml

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

### ingress安装
```shell

kubectl apply -f crd.yaml
kubectl apply -f rbac.yaml 

kubectl apply -f config.yaml -n kube-system

# 我们指定分配到master节点
kubectl apply -f deployment.yaml -n kube-system

kubectl apply -f dashboard.yaml -n kube-system

# 查看
kubectl get pods -n kube-system -l k8s-app=traefik -o wide

# 浏览器访问
http://traefik.mmc/dashboard/#/
```

### 参考资料
+ [《使用kubeadm安装kubernetes_v1.18.x》](https://kuboard.cn/install/install-k8s.html)
+ [《Kubernetes 私有集群负载均衡器终极解决方案》](https://www.modb.co/db/24870)
+ [《Traefik 2》](http://www.mydlq.club/article/41/)