# kiali

本地调试



首先下载kiali 和kiali-ui

## Kiali-ui

编译生成build文件夹



## kiali

### 配置启动参数

设置配置文件路径
```
-config /kiali/config.yaml
```



### 配置环境变量

```
KUBERNETES_SERVICE_HOST=10.1.11.76;
KUBERNETES_SERVICE_PORT=6443
```



### 设置配置文件

```
server:
  address: localhost
  port: 8000
  # 用户名密码
  credentials:
    username: admin
    passphrase: 123456
  # 静态页面路径
  static_content_root_directory: /Users/shenkonghui/src/git/kiali-ui/build
auth:
  # 使用用户名密码登录
  strategy:  login
in_cluster: false
login_token:
  expiration_seconds: 86400
  signing_key: secret:kiali-signing-key:key
```



### 修改代码

client.go 文件

```
func ConfigClient() (*rest.Config, error) {

	return clientcmd.BuildConfigFromFlags("", "/Users/shenkonghui/.kube/config")
}

```

token.go

```
func GetKialiToken() (string, error) {
	rest, err := clientcmd.BuildConfigFromFlags("", "/Users/shenkonghui/.kube/config")
	return rest.BearerToken, err
}
```

cache.go

```
func NewKialiCache() (KialiCache, error) {
cacheToken := config.BearerToken
}
```

