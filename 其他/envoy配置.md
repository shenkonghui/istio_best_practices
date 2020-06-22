# envoy配置

## 基本配置

#### admin配置

```text
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }
```

#### 监听器 + 路由

```text
static_resources:
  ## 监听
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 10000 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          stat_prefix: ingress_http
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: some_service }
          http_filters:
          - name: envoy.router
```

Listener

- name
- address
- filter_chains
  内置envoy.client_ssl_auth、envoy.echo、enovy.http_connection_manager、envoy.mongo_proxy、envoy.rate_limit、enovy.redis_proxy、envoy.tcp_proxy、http_filters、thrift_filters等等

route

- name
- virtual_hosts
  - name
  - domains
- Routes
  - match
  - route

#### 集群和目标地址

```text
  clusters:
  - name: some_service
    type: LOGICAL_DNS
    connect_timeout: 2s
    load_assignment:
      cluster_name: some_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: www.baidu.com
                port_value: 80
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: www.163.com
                port_value: 80
```

clusters

- name
  下游集群名，可定义一或多个
- connect_timeout 
  连接上游的超时时长，可带单位，如“0.25s”表示250毫秒
- type:集群类型
  解析集群的服务发现、如STATIC、STRICT_DNS、LOGICAL_DNS和EDS等
- lb_policy:
  负载均衡策略，ROUND_ROBIN(轮训)、LEAST_REQUEST(请求最少)、RING_HASH(环形hash)、RANDOM(随机)、MAGLEV(一致性hash)、CLUSTER_PROVIDED(定制)
- load_assignment
  type为STATIC、STRICT_DNS和LOGICAL_DNS时
- eds_cluster_config
  如果type为EDS则使用eds_cluster_config

## 高级配置

#### 故障注入

###### 延迟注入

```text
          http_filters:
          - name: envoy.fault
            config:
              delay:
                type: fixed
                fixed_delay: 10s
                percentage:
                  numerator: 100
                  denominator: HUNDRED
          - name: envoy.router
            config: {}
```

###### 异常注入

```text
          http_filters:
          - name: envoy.fault
            config:
              abort:
                http_status: 503 # 响应码
                percentage:
                  numerator: 10 # 故障比例为10/hundred
                  denominator: HUNDRED
          - name: envoy.router
            config: {}
```

#### 动态配置

## demo

#### 百度

```text
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      protocol: TCP
      address: 127.0.0.1
      port_value: 9901
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        protocol: TCP
        address: 0.0.0.0
        port_value: 10000
    filter_chains: #过滤器列表
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  host_rewrite: www.baidu.com
                  cluster: service_google
          http_filters:
          - name: envoy.filters.http.router
  clusters:
  - name: service_google
    connect_timeout: 0.25s
    type: LOGICAL_DNS
    # Comment out the following line to test on v6 networks
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_google
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: www.baidu.com
                port_value: 80
```

## 问题

host是老版本的配置就、将被废弃

Error: non-terminal filter named envoy.fault of type envoy.filters.http.fault is the last filter in a http filter chain.
要加 空的个

```text
          - name: envoy.router
            config: {}
```