## K8S部署Jenkins

### NFS创建
#### CentOS 7.6
```shell
# 目标机器 172.17.20.103
yum install nfs-utils rpcbind -y

mkdir -p /data1/k8s-vloume
echo "/data1/k8s-vloume  *(rw,no_root_squash,sync)" > /etc/exports
systemctl start rpcbind
systemctl enable rpcbind
systemctl enable nfs
systemctl restart nfs

# 测试
mkdir /home/frank/nfs -p
mount -t nfs 172.17.20.103:/data1/k8s-vloume /home/frank/nfs
umount -v /home/frank/nfs
```

#### Ubuntu 18.04
```shell
sudo apt-get install nfs-kernel-server

echo "/data1/k8s-vloume  *(rw,no_root_squash,sync)" > /etc/exports


sudo mkdir -p /data1/k8s-vloume
sudo chmod -R 777 /data1/k8s-vloume
sudo chown frank:frank /data1/k8s-vloume -R   # ipual 为当前用户，-R 表示递归更改该目录下所有文件

systemctl enable nfs-kernel-server
systemctl restart nfs-kernel-server

mkdir /home/frank/nfs -p
mount -t nfs 172.20.99.13:/data1/k8s-vloume /home/frank/nfs
umount -v /home/frank/nfs
```

### 创建namespace
```shell
kubectl create namespace kube-ops
```

### 目录挂在到PVC
+ 容器的 /var/jenkins_home 目录挂载到了一个名为 opspvc 的 PVC 对象上面.
+ 详情查看[jenkins_pvc.yaml](./jenkins_pvc.yaml), 其中nfs设置为上面的配置。
+ 创建pvc对象
    ```shell
    kubectl create -f jenkins_pvc.yaml
    ```

### Jenkins权限控制
+ 详情查看[rabc.yaml](./rabc.yaml)
+ 创建rabc相关资源
    ```shell
    kubectl create -f rbac.yaml
    ```

### Deployment
+ 部署服务，详情查看[deployment](./deployment.yaml)
+ 创建deployment资源
    ```shell
    kubectl create -f deployment.yml
    ```
+ 查看
    ```shell
    kubectl get all -n kube-ops
    ```

### Jenkins服务
+ Jenkins虽然已经起起来了，但是并不能通过浏览器访问，因为还没对外暴露，所以需要创建一个service(service.yml)
+ 我们这里通过 NodePort 的形式来暴露 Jenkins 的 web 服务，固定为30002端口，另外还需要暴露一个 agent 的端口，这个端口主要是用于 Jenkins 的 master 和 slave 之间通信使用的。
+ 详情查看[jenkins.yaml](./jenkins.yaml)
+ 创建Jenkins服务
    ```shell
    kubectl create -f service.yaml

    # 查看pod状态
    kubectl get pods -n kube-ops
    kubectl decribe pod  -f jenkins-85f6b6c6db-fb4z2 -n kube-ops
    kubectl logs -f jenkins-85f6b6c6db-fb4z2 -n kube-ops
    ```
### 访问Jenkins
+ 浏览器访问172.17.20.103:30002
+ 获取Jenkins initialAdminPassword
```shell  
kubectl logs -f jenkins-85f6b6c6db-fb4z2 -n kube-ops

# 日志里面可以看到，类似下面
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

b30d0a099a4f4727838dc3ce4e2e27df

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
```


**注** Jenkins可以通过流水器生成器(Snippet Generator)协助生成代码，这个功能非常实用。

### Jenkins PipeLine & k8s
+ [《基于 Jenkins 的 CI/CD (二)》](https://www.qikqiak.com/k8s-book/docs/37.Jenkins%20Pipeline.html)
+ 安装k8s插件，支持docker in docker模式，通过k8s动态调度Jenkins打包任务。

### 参考资料
+ [《基于 Jenkins 的 CI/CD (一)》](https://www.qikqiak.com/k8s-book/docs/36.Jenkins%20Slave.html)
+ [《Kubernetes下Jenkins CI的搭建》](https://hyrepo.com/tech/kubernetes-jenkins/)
+ [《Jenkins pipeline脚本编写实践分享》](https://zhuanlan.zhihu.com/p/51533506)
+ [《pipeline-examples》](https://github.com/jenkinsci/pipeline-examples)
+ [《pipeline for go》](https://bmuschko.com/blog/go-on-jenkins/)
+ [《jenkinsci/docker》](https://github.com/jenkinsci/docker)