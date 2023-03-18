# kubernetes-sample

## Pod

### Pod + ClusterIP
$ kubectl apply -f basic/pod-clusterip.yaml
pod/mynginx unchanged
service/mynginx-svc unchanged

$ kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
mynginx-svc   ClusterIP   10.97.219.125   <none>        8080/TCP   14m

$ kubectl apply -f basic/curl-pod.yaml
pod/curl unchanged

$ kubectl logs curl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

<title>Welcome to nginx!</title>
</body>
</html>
100   615  100   615    0     0  10818      0 --:--:-- --:--:-- --:--:-- 11388
