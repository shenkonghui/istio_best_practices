# Kiali-operator

kiali 支持operator的方式安装，比较新的版本kiali目前官网主要以operator的方式安装为主。

https://operatorhub.io/operator/kiali



## Install on Kubernetes

1. 安装operator生命周期管理器（OLM），该工具可帮助管理群集上运行的operator。
 ```yaml
   curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/0.15.1/install.sh | bash -s 0.15.1
 ```

2. 运行如下命令 安装kiali-operator

``` yaml
kubectl create -f https://operatorhub.io/install/kiali.yaml

```

3. 该operator将安装在“operator”名称空间中，并且可以在群集中的所有名称空间中使用。
```
kubectl get csv -n operators
```

如果看不到，则需要修改crd中的镜像

```
kubectl edit CatalogSource  operatorhubio-catalog   -n olm
```

4. 安装cr

   先切换到你想要安装的namespace，比如istio-system
   
   官网有很多kiali的cr文件，https://github.com/kiali/kiali-operator/tree/master/deploy/kiali

```
kubectl apply -f kiali/kiali_cr_minimal.yaml
```

```
 ~/tmp  kubectl get deployment |grep kiali
kiali            1/1     1            1           5m50s
```

 