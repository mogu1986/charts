# charts 0.1
## 1. 功能列表:
- 此模版部署所有xxx前后端工程
- 集成 arthas3.1.7
- 集成 skywalking7.0.0，默认关闭
- 集成 filebeat7.5.1，环境变量注入了节点ip，默认关闭
- 集成 Prometheus监控，见service上prometheus.io/scrape注解，默认不允许发现
- 加入调度策略，默认不允许部署在ops节点，集群部署分布在不同节点，不同可用区

## 2. 用户指南（如何部署）:
### 2.0 添加密钥
```bash
kubectl create secret generic encrypt --from-env-file=encrypt -n uat
```

#### 2.1 添加仓库
```bash
# 添加仓库并更新
helm repo add meiwu https://xxx.tencentcloudcr.com/chartrepo/library --username xxx --password xxx 
helm repo up
```
#### 2.2 部署java工程(默认不创建service，99%工程都是经过gateway外网访问)
```bash
helm install distribution --namespace prod --version 0.1 meiwu/meiwu \
--set nameOverride=distribution \
--set image.repository=xxx.tencentcloudcr.com/library/distribution \
--set image.tag=116 \
--set skywalking.enabled=false \                     # 是否开启skywalking, prod开启
--set log.enabled=false \                            # 是否开启日志采集，prod开启
--set log.elasticsearch.host=192.168.48.45 \
--set log.elasticsearch.username=elastic \
--set log.elasticsearch.password=12345 \
--set resources.limits.cpu=2 \
--set resources.limits.memory=8Gi \
--set resources.requests.cpu=2 \
--set resources.requests.memory=8Gi \
--set spring.profiles.active=prod \
--set apollo.configservice=http://apollo.xxx.cn \  # dev、test用此配置，uat、prod下删除
--set apollo.meta=http://apollo-config-server-prod.apollo:8080 \    # uat、prod下用此配置，dev、test删除
--set service.enabled=true \                         # 只有mw-iim，gateway 需要加
--set ingress.enabled=true \                         # 只有mw-iim，gateway 需要加
--set ingress.hosts[0]=distribution.test.mw \        # 只有mw-iim，gateway 需要加
--set ingress.tls[0].hosts[0]=$host \                # 只有mw-iim，gateway 需要加
--set ingress.tls[0].secretName=ssl                  # 只有mw-iim，gateway 需要加
```

#### 2.3 部署前端工程(前端工程要传递 lang=js 参数)
```bash
helm install nginx --namespace test --version 0.1 meiwu/meiwu \
--set nameOverride=nginx \
--set image.repository=xxx.tencentcloudcr.com/library/nginx \
--set image.tag=1.9 \
--set lang=js \
--set resources.limits.cpu=4 \
--set resources.limits.memory=8Gi \
--set resources.requests.cpu=4 \
--set resources.requests.memory=8Gi \
--set service.enabled=true \
--set ingress.enabled=true \
--set ingress.hosts[0]=nginx.test.com \
--set ingress.tls[0].hosts[0]=nginx.test.com \
--set ingress.tls[0].secretName=ssl
```

## 3. 开发指南:

### 3.1 推送私服:
```bash
helm push ./meiwu meiwu
```

### 3.2 调试:
```bash
helm install distribution --namespace prod --set skywalking.enabled=true --set log.enabled=true --set spring.profiles.active=dev --set service.enabled=true --dry-run --debug ./meiwu
```

## 附：参数配置 : 
参数 | 描述 | 默认值 | 是否必须
---|---|---|---
nameOverride|名字|-|必须设置
lang|区分前后端工程,可以选择java、js|java|必须设置
skywalking.enabled|是否开启skywalking|false|-
arms.enabled|是否开启arms监控|false|-
ports.http.enabled|是否开启http(80)端口|true|-
ports.xxl.enabled|是否开启xxljob(9991)端口|false|-
replicaCount|部署实例个数|1|-
image.repository|镜像仓库|harbor.top.mw/library/distribution|必须设置
image.tag|版本号|5|必须设置
imagePullSecrets|拉去镜像的密钥|[]|-
log.enabled|是否采集日志|false|-
log.path|采集日志目录|/log|-
log.elasticsearch.host|es地址|-|-
log.elasticsearch.port|es端口|9200|-
log.elasticsearch.username|es账号|-|-
log.elasticsearch.password|es密码|-|-
service.enabled|是否开启svc|false|前端工程必须设置，需要暴露的工程必须设置
ingress.enabled|是否开启ingress|false|依赖service.endabled, 需要暴露的工程必须设置
ingress.hosts[0]|外网域名|-|-
