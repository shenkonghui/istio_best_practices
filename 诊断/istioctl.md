# istioctl 工具

istioctl是istio cli工具，可以实现以下功能

- 集群安装、卸载、
- 手动注入sider到普通的yaml文件中
- 分析istio配置
- 开启dashboard观察网络
- 开启controlz dashboard 自检
- 手动注册/注销服务
- 查看监控指标
- ...

更多配置详情请查看官方文档

https://istio.io/latest/zh/docs/reference/commands/istioctl/



## 分析配置

可以分析当前集群的配置，也可以分析配置文件

- service port name 检查
- namespace enabled  injection检查
- 其他 可以通过该命令查看  istioctl analyze -L

```
// 分析集群中命名空间为default
istioctl analyze -n default
// 分析集群中的所有命名空间
istioctl analyze --all-namespaces

// 分析gateway.yaml文件
istioctl analyze gateway.yaml
// 分析
istioctl analyze bookinfo/
```



## 描述网络

查看每个pod的网络状态

如果输出列表中缺少某个代理则意味着它当前未连接到 Polit 实例，所以它无法接收到任何配置。此外，如果它被标记为 stale，则意味着存在网络问题或者需要扩展 Pilot。

```
[root@host-76 shenkonghui]# istioctl proxy-status 
NAME                                                   CDS        LDS        EDS        RDS          PILOT                       VERSION
details-v1-78d78fbddf-rr56d.istio-test                 SYNCED     SYNCED     SYNCED     SYNCED       istiod-778ddcd9c7-5ngtt     1.6.3
httpbin-56f87c9c66-kzhcs.foo                           SYNCED     SYNCED     SYNCED     SYNCED       istiod-778ddcd9c7-5ngtt     1.6.3
istio-egressgateway-79df5cd489-85zsq.istio-system      SYNCED     SYNCED     SYNCED     NOT SENT     istiod-778ddcd9c7-5ngtt     1.6.3
```



## k8s ingress 转化成istio gateway

`istioctl convert-ingress -f [xxx.yaml]`



## 分析istio配置

可以查看以下配置

- 服务注册(envoy实例)情况

  `istioctl proxy-config cluster <pod-name> [flags]`

- envoy的引导配置

  `istioctl proxy-config bootstrap <pod-name> [flags]`

- 监听器(listener)配置

  `istioctl proxy-config listener <pod-name> [flags]`

- 路由(route)配置

  `istioctl proxy-config route <pod-name> [flags]`

- 端点(endpoints)配置

  `istioctl proxy-config endpoints <pod-name> [flags]`

检索集群中注册的服务情况

### 服务注册(envoy实例)情况
```
[root@host-76 /]# istioctl proxy-config cluster <pod-name> [flags]
SERVICE FQDN                                                         PORT      SUBSET          DIRECTION     TYPE
BlackHoleCluster                                                     -         -               -             STATIC
InboundPassthroughClusterIpv4                                        -         -               -             ORIGINAL_DST
PassthroughCluster                                                   -         -               -             ORIGINAL_DST
agent                                                                -         -               -             STATIC
calico-rest.kube-system.svc.cluster.local                            38080     -               outbound      EDS
details.istio-test.svc.cluster.local                                 9080      http            inbound       STATIC
...
```
### envoy的引导配置
```
[root@host-76 shenkonghui]# istioctl proxy-config bootstrap details-v1-78d78fbddf-rr56d -n istio-test
{
    "bootstrap": {
        "node": {
            "id": "sidecar~10.168.244.47~details-v1-78d78fbddf-rr56d.istio-test~istio-test.svc.cluster.local",
            "cluster": "details.istio-test",
            "metadata": {
                    "APP_CONTAINERS": "[\n    details\n]",
                    "CLUSTER_ID": "Kubernetes",
                    "CONFIG_NAMESPACE": "istio-test",
                    "EXCHANGE_KEYS": "NAME,NAMESPACE,INSTANCE_IPS,LABELS,OWNER,PLATFORM_METADATA,WORKLOAD_NAME,MESH_ID,SERVICE_ACCOUNT,CLUSTER_ID",
                    "INSTANCE_IPS": "10.168.244.47",
                    "INTERCEPTION_MODE": "REDIRECT",
                    "ISTIO_PROXY_SHA": "istio-proxy:9085616b434c0e0b54cc677cfd3b97a0fed71c0c",
                    "ISTIO_VERSION": "1.6.3",
                    ...
                    }
```

