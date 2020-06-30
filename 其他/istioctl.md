# istioctl



## 分析配置

可以分析当前集群的配置，也可以分析配置文件

- port name 检查
- namespace enabled  injection检查

```
// 分析集群中的所有命名空间
istioctl analyze --all-namespaces
// 分析gateway.yaml文件
istioctl analyze gateway.yaml
// 分析
istioctl analyze bookinfo/
```

