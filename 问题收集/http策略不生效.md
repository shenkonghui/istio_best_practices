# http 策略不生效

尝试设置故障注入，就是没有生效。

#### 结论

Service应提供“app”标签（供请求跟踪特性使用），而且服务定义中的“spec.ports.name”必须正确命名（http、http2、grpc、redis、mongo），否则，Envoy将把该服务流量作为普通的TCP服务对待，你将无法在那些服务中使用7层特性！