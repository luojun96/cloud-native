
# Microservice

## 运维

## 开发

### 灰度发布

**灰度，就是存在于黑与白之间的一个平滑过渡的区域**。对于互联网产品来说，上线和未上线就是黑与白之分，而实现未上线功能平稳过渡的一种方式就叫做灰度发布。

- **按流量百分比**
  - 先到先得得方式，比如：限制10%的用户体验的事新版本，90%的用户体验的是老版本。先访问网站的用户就优先命中新版本，直到流量用完为止。
- **按人群划分**
  - 按用户ID，用户IP，设备类型划分。
    - 可通过平时的埋点上报数据得知用户的PV、UV、页面平均访问时长等数据，根据用户活跃度来让用户优先体验新版本，进而快速观察使用效果。
  - 按地域，性别，年龄等用户画像划分。
    - 可通过用户的性别、年龄等做下新版本的对比效果老看看目标用户在新版本的使用年龄段、性别范围是多少。

### 最佳实践

- **为客户端添加超时时间**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - timeout: 10s
    route:
    - destination:
        host: reviews
        subset: v1
```

- **加断路器规则**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 5
      interval: 5s
      baseEjectionTime: 30s
      maxEjectionPercent: 10
```
