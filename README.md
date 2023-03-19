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
-- ingress contoller 버젼 변경을 통해서 해결함(0.44.0 -> 1.3.1) => 여기서부터 참고하자

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
ingress-nginx-controller             LoadBalancer   10.97.71.217    <pending>     80:30096/TCP,443:30309/TCP   11m
ingress-nginx-controller-admission   ClusterIP      10.103.237.19   <none>        443/TCP                      11m

-- LoadBalancer도 VM IP를 통해서 되는 걸 보면 NodePort기능도 적용이 된다. External IP는 클라우드 서비스나 MetalLB같은 로드벨런서가 있어야 한다.
jaekwanjeon@kubernetes-master:~/arch-k8s-sample/kubernetes-sample$ curl 172.17.0.1:30096
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

### Istio 설치

#### 설치 예제(Istio 제공 BookInfo 샘플)
```
$ mkdir ~/bin
$ cd bin
$ curl -L https://istio.io/downloadIstio | sh -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   102  100   102    0     0    286      0 --:--:-- --:--:-- --:--:--   286
100  4856  100  4856    0     0   6331      0 --:--:-- --:--:-- --:--:--  6331

Downloading istio-1.17.1 from https://github.com/istio/istio/releases/download/1.17.1/istio-1.17.1-linux-amd64.tar.gz ...
Istio 1.17.1 Download Complete!

Istio has been successfully downloaded into the istio-1.17.1 folder on your system.

Next Steps:
See https://istio.io/latest/docs/setup/install/ to add Istio to your Kubernetes cluster.

To configure the istioctl client tool for your workstation,
add the /home/jaekwanjeon/bin/istio-1.17.1/bin directory to your environment path variable with:
	 export PATH="$PATH:/home/jaekwanjeon/bin/istio-1.17.1/bin"

Begin the Istio pre-installation check by running:
	 istioctl x precheck 

Need more information? Visit https://istio.io/latest/docs/setup/install/ 


$ cd istio-1.17.1
$ export PATH=$PWD/bin:$PATH

$ istioctl profile list
Istio configuration profiles:
    ambient
    default
    demo
    empty
    external
    minimal
    openshift
    preview
    remote

$ istioctl install --set profile=demo -y
 Istio core installed                                                          
- Processing resources for Istiod. Waiting for Deployment/istio-system/istiod   
✔ Istiod installed                                                                                                        
✔ Egress gateways installed                                                                                               
✔ Ingress gateways installed                                                                                              
✔ Installation complete                                                                                                   Making this installation the default for injection and validation.

Thank you for installing Istio 1.17.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/hMHGiwZHPU7UQRWe9

$ kubectl label namespace default istio-injection=enabled
namespace/default labeled

$ kubectl get ns -L istio-injection
NAME              STATUS   AGE   ISTIO-INJECTION
covid             Active   30h   
default           Active   11d   enabled
istio-system      Active   49m   
kube-flannel      Active   11d   
kube-node-lease   Active   11d   
kube-public       Active   11d   
kube-system       Active   11d   
login             Active   11d   


$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created

$ kubectl get services
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
details       ClusterIP   10.104.225.32    <none>        9080/TCP         1s
frontend      ClusterIP   10.108.89.59     <none>        80/TCP           8h
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP          11d
mynginx-svc   NodePort    10.97.37.148     <none>        8080:31001/TCP   9h
productpage   ClusterIP   10.102.189.189   <none>        9080/TCP         1s
ratings       ClusterIP   10.99.68.126     <none>        9080/TCP         1s
reviews       ClusterIP   10.96.141.2      <none>        9080/TCP         1s

$ kubectl get pods
NAME                          READY   STATUS     RESTARTS      AGE
details-v1-6997d94bb9-t26hc   0/2     Init:0/1   0             2s
frontend-66dccbdd6d-clr6b     1/1     Running    1 (29m ago)   8h
frontend-66dccbdd6d-d5782     1/1     Running    1 (29m ago)   8h
ratings-v1-b8f8fcf49-fmn9f    0/2     Init:0/1   0             1s

$kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"

$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created

$ istioctl analyze
✔ No validation issues found when analyzing namespace: default

$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
$ export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')

$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
$ echo "$GATEWAY_URL"
192.168.0.3:30468
```
![image](https://user-images.githubusercontent.com/3446997/226106623-799d187a-40b4-4886-b848-e8610b76e4b9.png)

#### 설치 예제(Istio 폴더)

```
$ kubectl apply -f ./istio/
$ kubectl apply -f .
deployment.apps/frontend unchanged
service/frontend-svc created
gateway.networking.istio.io/simple-istio-gateway unchanged
virtualservice.networking.istio.io/simple-vertual-service unchanged

$ istioctl analyze
✔ No validation issues found when analyzing namespace: default.

$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
jaekwanjeon@kubernetes-master:~/arch-k8s-sample/kubernetes-sample/istio$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}'
> )
$ export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
$ echo "$GATEWAY_URL"
192.168.0.3:30468
$ curl 192.168.0.3:30468
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
