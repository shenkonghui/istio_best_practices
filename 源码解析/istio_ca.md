# istio_ca


https://www.jianshu.com/p/b6ed6d2a86b4

查看所有secret, 可以每个serviceaccount都额外增加了一个类型为istio.io/key-and-cert的secret, 名字就是istio.加上serviceaccount的名字.

## 启动参数

```
--self-signed-ca --append-dns-names=true --grpc-port=8060 --citadel-storage-namespace=istio-system --custom-dns-names=istio-pilot-service-account.istio-system:istio-pilot.istio-system --monitoring-port=15014 --self-signed-ca=true --workload-cert-ttl=2160h
```


## 流程
主函数
1. 读取ca(istio-ca-secret)，如果没有则创建(certificate(ca-cert.pem)和private key(ca-key.pem))
2. 创建istio-security 



#### 核心代码流程整理

- main()
    - CreateClientset()
    - createCA()
        - if selfSignedCA
            - NewSelfSignedIstioCAOptions()
        - else
            - NewPluggedCertIstioCAOptions()
        - NewIstioCA()
        - if LivenessProbe()
            - NewLivenessCheckController()
            - livenessProbeChecker.Run()
    - if !serverOnly()
        - NewSecretController()
    - Run()
    - if Grpc 
        - GetIdentityRegistry()
        - AddMapping
        - NewServiceController
        - NewServiceAccountController
        - caserver.New()
    - if Monitor
        - monitoring.NewMonitor()
        - monitor.Start()
    - if cAClientConfig
      - caclient.NewKeyCertBundleRotator()


#### 创建ca

#### secret控制器

- NewSecretController()
    - NewInformer(&v1.ServiceAccount...) saAdded + saDeleted
    - NewInformer(&v1.Secret..) scrtDeleted + scrtUpdated
    -
    

监听事件处理

- saAdded()
    - upsertSecret()
            - BuildSecret()
    - generateKeyAndCert()
    - for 3 Secrets().Create()
    
- saDeleted()
    
    - deleteSecret()
    
- scrtUpdated()
    - ParsePemEncodedCertificate()
    - refreshSecret()
    
- scrtDeleted()

    


secret更新
1. secret更新事件

2. 获取secret中的cert-chain.pem  

3. 获得了PEM编码的证书

4. 验证证书是否到期

   


sa创建事件
1. 验证namespace是否启动istio
2. 获取sa
3. 生成证书签名和私钥
4. 生成证书
5. 创建secret