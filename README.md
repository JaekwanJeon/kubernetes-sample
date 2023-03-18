# kubernetes-sample

## Pod

### Pod + ClusterIP
```
$ kubectl apply -f basic/pod-clusterip.yaml
pod/mynginx unchanged
service/mynginx-svc unchanged

$ kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
mynginx-svc   ClusterIP   10.97.219.125   <none>        8080/TCP   14m

$ kubectl apply -f basic/curl-pod.yaml
pod/curl unchanged
$ kubectl logs curl
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### Pod + NodePort
```
$ kubectl apply -f basic/pod-nodeport.yaml
pod/mynginx created
service/mynginx-svc created

$ kubectl get svc
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
mynginx-svc   NodePort    10.97.37.148   <none>        8080:31001/TCP   7s

$ ifconfig
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255

$ curl 172.17.0.1:31001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
  
## Nginx Ingress
### Nginx Ingress 설치
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/cloud/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created



$ kubectl get svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.107.167.157   <pending>     80:31852/TCP,443:32238/TCP   51s
ingress-nginx-controller-admission   ClusterIP   10.99.73.175     <none>        443/TCP                      51s
-- LoadBalaner를 NodePort로 변경
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/baremetal/deploy.yaml
namespace/ingress-nginx unchanged
serviceaccount/ingress-nginx unchanged
configmap/ingress-nginx-controller configured
clusterrole.rbac.authorization.k8s.io/ingress-nginx unchanged
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx unchanged
role.rbac.authorization.k8s.io/ingress-nginx unchanged
rolebinding.rbac.authorization.k8s.io/ingress-nginx unchanged
service/ingress-nginx-controller-admission unchanged
service/ingress-nginx-controller configured
deployment.apps/ingress-nginx-controller configured
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission configured
serviceaccount/ingress-nginx-admission unchanged
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
role.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
job.batch/ingress-nginx-admission-create unchanged
job.batch/ingress-nginx-admission-patch unchanged

$ kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.107.167.157   <none>        80:31852/TCP,443:32238/TCP   117s
ingress-nginx-controller-admission   ClusterIP   10.99.73.175     <none>        443/TCP                      117s

$ kubectl apply -f ingress/
deployment.apps/frontend created
service/frontend created
ingress.networking.k8s.io/covid19-ingress created

$ kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.107.167.157   <none>        80:31852/TCP,443:32238/TCP   117s
ingress-nginx-controller-admission   ClusterIP   10.99.73.175     <none>        443/TCP                      117s

$ ifconfig
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        
$ curl 172.17.0.1:31852
curl: (7) Failed to connect to 172.17.0.1 port 31852: 연결이 거부됨
-- 이런 경우는 nginx ingress를 확인해 보자 (CrashLoopBackOff)
$ kubectl get all -A
NAMESPACE       NAME                                            READY   STATUS             RESTARTS        AGE
default         pod/frontend-66dccbdd6d-clr6b                   1/1     Running            0               8m34s
default         pod/frontend-66dccbdd6d-d5782                   1/1     Running            0               8m34s
default         pod/mynginx                                     1/1     Running            0               64m
ingress-nginx   pod/ingress-nginx-admission-create-ct2jw        0/1     Completed          0               12m
ingress-nginx   pod/ingress-nginx-controller-56bfb6cf77-rvb8h   0/1     CrashLoopBackOff   6 (71s ago)     10m
ingress-nginx   pod/ingress-nginx-controller-6548f6f589-kv24v   0/1     CrashLoopBackOff   6 (2m39s ago)   12m
kube-flannel    pod/kube-flannel-ds-k6qqb                       1/1     Running            3 (3h3m ago)    10d
kube-system     pod/coredns-787d4945fb-8qn5j                    1/1     Running            3 (3h3m ago)    10d
kube-system     pod/coredns-787d4945fb-xpl5g                    1/1     Running            3 (3h3m ago)    10d
kube-system     pod/etcd-kubernetes-master                      1/1     Running            11 (3h3m ago)   10d
kube-system     pod/kube-apiserver-kubernetes-master            1/1     Running            11 (3h3m ago)   10d
kube-system     pod/kube-controller-manager-kubernetes-master   1/1     Running            3 (3h3m ago)    10d
kube-system     pod/kube-proxy-sk96f                            1/1     Running            3 (3h3m ago)    10d
kube-system     pod/kube-scheduler-kubernetes-master            1/1     Running            11 (3h3m ago)   10d
login           pod/postgres-57d545847b-m5t9b                   1/1     Running            3 (3h3m ago)    10d

NAMESPACE       NAME                                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
default         service/frontend                             ClusterIP   10.108.89.59     <none>        80/TCP                       8m34s
default         service/kubernetes                           ClusterIP   10.96.0.1        <none>        443/TCP                      10d
default         service/mynginx-svc                          NodePort    10.97.37.148     <none>        8080:31001/TCP               64m
ingress-nginx   service/ingress-nginx-controller             NodePort    10.107.167.157   <none>        80:31852/TCP,443:32238/TCP   12m
ingress-nginx   service/ingress-nginx-controller-admission   ClusterIP   10.99.73.175     <none>        443/TCP                      12m
kube-system     service/kube-dns                             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       10d
login           service/postgres                             ClusterIP   10.107.78.62     <none>        5432/TCP                     10d

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   10d
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   10d

NAMESPACE       NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
default         deployment.apps/frontend                   2/2     2            2           8m34s
ingress-nginx   deployment.apps/ingress-nginx-controller   0/1     1            0           12m
kube-system     deployment.apps/coredns                    2/2     2            2           10d
login           deployment.apps/postgres                   1/1     1            1           10d

NAMESPACE       NAME                                                  DESIRED   CURRENT   READY   AGE
default         replicaset.apps/frontend-66dccbdd6d                   2         2         2       8m34s
ingress-nginx   replicaset.apps/ingress-nginx-controller-56bfb6cf77   1         1         0       10m
ingress-nginx   replicaset.apps/ingress-nginx-controller-6548f6f589   1         1         0       12m
kube-system     replicaset.apps/coredns-787d4945fb                    2         2         2       10d
login           replicaset.apps/postgres-57d545847b                   1         1         1       10d

NAMESPACE       NAME                                       COMPLETIONS   DURATION   AGE
ingress-nginx   job.batch/ingress-nginx-admission-create   1/1           14s        12m
ingress-nginx   job.batch/ingress-nginx-admission-patch    0/1           12m        12m

-- ingress contoller 버젼 변경을 통해서 해결함(0.44.0 -> 1.3.1)
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/baremetal/deploy.yaml
namespace/ingress-nginx unchanged
serviceaccount/ingress-nginx unchanged
serviceaccount/ingress-nginx-admission unchanged
role.rbac.authorization.k8s.io/ingress-nginx unchanged
role.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
clusterrole.rbac.authorization.k8s.io/ingress-nginx unchanged
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
rolebinding.rbac.authorization.k8s.io/ingress-nginx unchanged
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx unchanged
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
configmap/ingress-nginx-controller unchanged
service/ingress-nginx-controller unchanged
service/ingress-nginx-controller-admission unchanged
deployment.apps/ingress-nginx-controller configured
job.batch/ingress-nginx-admission-create unchanged
job.batch/ingress-nginx-admission-patch unchanged
ingressclass.networking.k8s.io/nginx unchanged
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission configured

$ kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.110.209.233   <none>        80:30956/TCP,443:31964/TCP   2m52s
ingress-nginx-controller-admission   ClusterIP   10.108.221.178   <none>        443/TCP                      2m52

$ ifconfig
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        
$ curl 172.17.0.1:30956
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### Ingress 및 Pod 설치(LoadBalancer)
https://mlops-for-all.github.io/docs/appendix/metallb/
```
$ kubectl apply -f ingress/
$ kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/baremetal/deploy.yaml
namespace "ingress-nginx" deleted
serviceaccount "ingress-nginx" deleted
serviceaccount "ingress-nginx-admission" deleted
role.rbac.authorization.k8s.io "ingress-nginx" deleted
role.rbac.authorization.k8s.io "ingress-nginx-admission" deleted
clusterrole.rbac.authorization.k8s.io "ingress-nginx" deleted
clusterrole.rbac.authorization.k8s.io "ingress-nginx-admission" deleted
rolebinding.rbac.authorization.k8s.io "ingress-nginx" deleted
rolebinding.rbac.authorization.k8s.io "ingress-nginx-admission" deleted
clusterrolebinding.rbac.authorization.k8s.io "ingress-nginx" deleted
clusterrolebinding.rbac.authorization.k8s.io "ingress-nginx-admission" deleted
configmap "ingress-nginx-controller" deleted
service "ingress-nginx-controller" deleted
service "ingress-nginx-controller-admission" deleted
deployment.apps "ingress-nginx-controller" deleted
job.batch "ingress-nginx-admission-create" deleted
job.batch "ingress-nginx-admission-patch" deleted
ingressclass.networking.k8s.io "nginx" deleted

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.1/deploy/static/provider/cloud/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created

$ kubectl get svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.97.71.217    <pending>     80:30096/TCP,443:30309/TCP   45s
ingress-nginx-controller-admission   ClusterIP      10.103.237.19   <none>        443/TCP                      44s


```
