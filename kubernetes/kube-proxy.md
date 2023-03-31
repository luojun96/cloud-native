## kube-proxy example of useing iptables to do load balancing
### Objects
nginx deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
```
nginx service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-basic
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: ClusterIP
```
### Running data in cluster
pod and services in cluster:

```
nginx-deployment-748c667d99-9xhr6   1/1     Running   0          5h20m   10.0.1.218   work   <none>           <none>
nginx-deployment-748c667d99-p74hc   1/1     Running   0          5h20m   10.0.0.128   code   <none>           <none>

nginx-basic   ClusterIP   10.96.158.136   <none>        80/TCP    7h24m   app=nginx
```
the expected target mapping of service to pods:

```
10.96.158.136:80 =>
10.0.1.218:80
10.0.0.128:80
```
iptables in nodes:```iptables-save -t nat``` or ```iptables -L -t nat```

```
...
:KUBE-SERVICES - [0:0]
...
-A PREROUTING -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A OUTPUT -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
...
-A KUBE-SERVICES -d 10.96.158.136/32 -p tcp -m comment --comment "default/nginx-basic:http cluster IP" -m tcp --dport 80 -j KUBE-SVC-WWRFY3PZ7W3FGMQW
...
-A KUBE-SVC-WWRFY3PZ7W3FGMQW -m comment --comment "default/nginx-basic:http -> 10.0.0.128:80" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-PM2T4OW2YNAS2OJ4
-A KUBE-SEP-PM2T4OW2YNAS2OJ4 -p tcp -m comment --comment "default/nginx-basic:http" -m tcp -j DNAT --to-destination 10.0.0.128:80

-A KUBE-SVC-WWRFY3PZ7W3FGMQW -m comment --comment "default/nginx-basic:http -> 10.0.1.218:80" -j KUBE-SEP-HS7I6HCG4KP2FFMJ
-A KUBE-SEP-HS7I6HCG4KP2FFMJ -p tcp -m comment --comment "default/nginx-basic:http" -m tcp -j DNAT --to-destination 10.0.1.218:80
```

