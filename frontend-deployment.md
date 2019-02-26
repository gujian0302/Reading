### 前端项目部署


## 登录

```
docker login registry.cn-shanghai.aliyun.com
##接下来命令行提示输出账号和密码
```

## 重新打包docker-image并上传

根据Semantic Version重新定义version,并打包上传镜像

## 默认已经配置了kubectl
kubectl 滚动升级镜像

1 . kubectl set image deployment/admin image=admin-web:xx.xx.xx


