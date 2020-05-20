## K8S部署GoApp

### 编写Go App
#### 配置(Config)
    比较合适的配置管理是写入环境变量。
#### 日志(Logging)
    日志写入/dev/stdout或者/dev/stderr,确保日志成为容器日志的一部分。
#### 数据持久化(Data Persistency)
    只存储必要的数据。
#### 信号处理(Termination signals)  
    何为合适的处理？
#### 程序重启(Application Restart)
    通过杀死pod或者node测试服务。
#### 高可用(High Availability)
    水平扩展和节点维护。
#### 服务探测(Probes)
    开启Liveness和Readiness探测。

### Docker化
```shell
# 构建镜像
docker build -f Dockerfile -t docker-registry.dhc:5000/go-app:1.0.0 .

# 推送到镜像中心
docker push docker-registry.dhc:5000/go-app:1.0.0

```
#### K8S deployment
```shell
kubectl apply -f k8s-deployment.yml

# deploy已经使用了ingress-nginx方案 
kubectl get services
kubectl get pods
```

#### 使用ingress方案
```shell

# 以ingress方式部署
# kubectl apply -f ingress-app.yaml  
```


### 参考资料
+ [《Deploying a containerized Go app on Kubernetes》](https://www.callicoder.com/deploy-containerized-go-app-kubernetes/)
