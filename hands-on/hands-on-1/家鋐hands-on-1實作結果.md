# 家鋐hands-on-1實作結果


## :pencil: 環境建立:
執行環境: Windows10 家用版
Docker Desktop 4.4.4 (73704) 
minikube version: v1.25.1
### 執行指令
```bash=
$ minikube start --driver=docker
```
### 執行結果
```bash=
* minikube v1.25.1 on Microsoft Windows 10 Home 10.0.19042 Build 19042
* Using the docker driver based on existing profile
* Starting control plane node minikube in cluster minikube
* Pulling base image ...
* Restarting existing docker container for "minikube" ...
* Preparing Kubernetes v1.23.1 on Docker 20.10.12 ...
  - kubelet.housekeeping-interval=5m
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
### 確認kubenetes已開啟
```bash=
$ kubectl get all
```
### 執行結果
```bash=
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3m30s
```
---
## :pencil:建立專屬 namespace
### 指令
```bash=
$ kubectl create namespace esb22220
namespace/esb22220 created
```
### 確認新增之namespace
```bash=
$ kubectl get ns
NAME              STATUS   AGE
default           Active   6m26s
esb22220          Active   12s
kube-node-lease   Active   6m28s
kube-public       Active   6m28s
kube-system       Active   6m28s
```
### 更換現在 namespace
```bash=
$ kubectl config set-context --current --namespace=esb22220
Context "minikube" modified.
```
### 確認是否已經切換成功
:computer: windows cmd環境使用findstr取代grep
```bash=
$ kubectl config view --minify | findstr namespace:
   namespace: esb22220
```
---
## :pencil:CLI 基本操作

### 用 CLI 創建

用 CLI 建立一個 image 為 `nginx:latest` 的 pod

```bash=
$ kubectl run nginx --image=nginx:latest
pod/nginx created
```
### 用 `get` 查看 pod 資訊
```bash=
$ kubectl  get pod
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          9s
#正在建立...
#  -o wide 看更多資訊
$ kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          25s   172.17.0.3   minikube   <none>           <none>
```

### 用 `describe` 查看 pod 細節 

```bash=
$ kubectl describe pod nginx
Name:         nginx
Namespace:    esb22220
Priority:     0
Node:         minikube/192.168.49.2
....
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m7s   default-scheduler  Successfully assigned esb22220/nginx to minikube
  Normal  Pulling    3m6s   kubelet            Pulling image "nginx:latest"
  Normal  Pulled     2m57s  kubelet            Successfully pulled image "nginx:latest" in 8.5931333s
  Normal  Created    2m57s  kubelet            Created container nginx
  Normal  Started    2m57s  kubelet            Started container nginx
```

### 測試 `port-forward` 是否成功
* port-forward 透過 kubectl 在本地端建立與遠端 pod 的連線
```bash=
# nginx 跑在 80 port
$ kubectl port-forward pod/nginx 80:80
Forwarding from 127.0.0.1:80 -> 80
Forwarding from [::1]:80 -> 80
Handling connection for 80
Handling connection for 80
...
```
* 檢查網頁
```bash=
$ curl 127.0.0.1:80
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

## :pencil:`exec` 進去 pod 內操作 

* binary: `bash`
* 平常可以利用 exec 進到 pod 內除錯

```bash=
$ kubectl exec -it nginx bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@nginx:/# echo 'Hello World' > /usr/share/nginx/html/index.html
root@nginx:/# exit
exit
command terminated with exit code 130
```
* 檢查結果
```bash=
$ curl 127.0.0.1:80
Hello World
```
### 查看 pod log
```bash=
$ kubectl logs -f nginx
```

## :pencil:上標籤

### 用 CLI 給 pod 上標籤
```bash=
# 指令
$ kubectl label pod nginx key=nginx
pod/nginx labeled
```

### 用 editor 給 pod 上標籤

```bash=
$ kubectl edit pod nginx
#跳出textbook視窗，直接進行編輯
```

### 確認上標籤成功
```bash=
$ kubectl get pod --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          17m   key=nginx,run=nginx
#LABELS欄位可以看到多了一個剛剛命名key的label
```
### 移除標籤
* 最後有個 `-` 代表移除
```bash=
$ kubectl label pod nginx key-
pod/nginx labeled
```
* 檢查是否移除成功
```bash=
$ kubectl get pod --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          17m   run=nginx
#LEBELS欄位已移除key的label
```
## :pencil:Delete pod

```bash
# 用物件名刪除
$ kubectl delete pod nginx
pod "nginx" deleted
#檢查已移除
$ kubectl get pod
No resources found in esb22220 namespace.

# 因為有 graceful period，用 --force --grace-period=0 快速刪除 pod
# (--force --grace-period=0 僅作為測試，一般不建議使用)
$ kubectl delete pod <pod> --force --grace-period=0
```
# 除錯小技巧

假設今天要跑一個除錯用的 pod 且離開後就刪除，可以加上 `--rm -it`

```yaml
# 指令
$ kubectl run <pod> --generator=run-pod/v1 --rm -it --image=<image>｀ -- <binary>

# 範例
$ kubectl run foobar --generator=run-pod/v1 --rm -it --image=busybox -- sh
```

進到該 pod 的 shell 後就可以執行 ping 或是 nslookup 相關指令
