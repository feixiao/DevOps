## DevOps实践

### 版本
+  k8s v1.18.2
+  CentOS 7+

### 目录介绍
#### dns服务搭建
    dns服务搭建(ingress方案需要各自域名，通过域名转发到相应的服务)。

#### gitlab 
    基于docker部署gitlab, 方便管理go代码。

##### k8s
    部署k8s集群环境。

#### jenkins
+ 1: 基于k8s部署jenkins。
    + 1.1: 基于k8s调度jenkins slave的方式编译感觉太慢了。
        + 启动运行jenkins slave的pod
        + 下载go依赖库等  
+ 2: Docker in docker版本的Jenkins。 
+ 3: 基于Jenkins构建go项目pipeline。

#### app 
    k8s部署go app。

### 指导手册
+ [《12-Factor》](https://12factor.net/zh_cn/) 12-Factor 为构建SaaS 应用提供了方法论。
+ [《生产环境容器落地最佳实践》](https://studygolang.com/articles/27140)


#### 参考资料
+ [《Tutorials》](https://kubernetes.cn/docs/tutorials/)