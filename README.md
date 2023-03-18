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
```html
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


