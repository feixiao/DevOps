## Docker部署Jenkins

### 简介
+ 不在k8s中运行
+ 支持docker in docker
+ 支持操作k8s集群

### 部署
```shell
chown -R 1000:1000 /home/frank/volumes/jenkins_home
docker run -itd -p 18080:8080 -p 50000:50000 \
    --name="jenkins" \
    -v /home/frank/volumes/jenkins_home:/var/jenkins_home \
    -v $(which docker):/bin/docker \
    -v /var/run/docker.sock:/var/run/docker.sock \
    --restart unless-stopped \
    jenkins/jenkins:lts

# 密码在/var/jenkins_home/secrets/initialAdminPassword 即/home/frank/volumes/jenkins_home/secrets/initialAdminPassword
```

### 中国源
```
http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```



### 参考资料
+ [《搭建DevOps模式的项目》](https://juejin.im/post/5e40cf84518825490f721bfe)
+ [《Create A CI/CD Pipeline With Kubernetes And Jenkins》](https://www.magalix.com/blog/create-a-ci/cd-pipeline-with-kubernetes-and-jenkins)
