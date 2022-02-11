###### tags: `K8S`
# 家鋐hands on 2實作結果
- 執行環境: Windows10 家用版
Docker Desktop 4.4.4 (73704) 
minikube version: v1.25.1
## :pencil: Pod and Value
### 撰寫含有兩個容器的pod之YAML
:dart: 作業要求: 
pod中包含nginx及busybox兩個    container，並透過emptyDir share volume

* yaml內容實作:
    * 建立一個名稱為hands-on-2之pod
    * 於spec中設定兩個containers分別為==nginx==及==busybox==
    * 設定emptyDir之share-volume
    * 掛載nginx於/usr/share/nginx/html下
    * 掛載busybox到 /tmp，並利用 `watch echo 'Hands on 2 demo' > /tmp/index.html` 建立 index.html
### 透過yaml建立pod
1. 於yaml檔目錄開啟cmd後，確認kubenetes正確執行並於指定之namespace下
```bash=
$ kubectl config view --minify | findstr namespace:
   namespace: esb22220
```
2. 透過yaml建立pod
```bash=
$ kubectl apply -f pod.yaml
pod/hands-on-2 created
```
3. 檢查pod是否正常運行
```bash=
$ kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
hands-on-2   2/2     Running   0          10s
```
4. 取得部屬完之pod的yaml文件
```bash=
$ kubectl kubectl get pod hands-on-2 -o yaml > hands-on-2.yaml
```
- 可以看到文件中與原本上傳之yaml有許多更新，kubenetes自動補上了metadata中的annotation及一些timestamp。
```yaml=
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"nginx","version":"1"},"name":"hands-on-2","namespace":"esb22220"},"spec":{"containers":[{"image":"nginx:latest","name":"nginx","volumeMounts":[{"mountPath":"/usr/share/nginx/html","name":"share-volume"}]},{"command":["watch","echo 'Hands on 2 demo' \u003e /tmp/index.html"],"image":"busybox:latest","name":"busybox","volumeMounts":[{"mountPath":"/tmp","name":"share-volume"}]}],"volumes":[{"emptyDir":{},"name":"share-volume"}]}}
  creationTimestamp: "2022-02-11T03:17:33Z"
```
- 另外kubenetes也補上了節點相關資訊及這個pod目前的狀態紀錄
```yaml=
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2022-02-11T03:17:33Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2022-02-11T03:17:42Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2022-02-11T03:17:42Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2022-02-11T03:17:33Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://34ad8c276171a202607fac88d0d53880245a19196a04d69994e5b3b96295d1ac
    image: busybox:latest
    ...
    state:
      running:
        startedAt: "2022-02-11T03:17:42Z"
```
## :pencil: Service撰寫
* 透過部屬一個nginx service為剛剛的pod建立pod與之遠端連線
* yaml內容實作:
    * 建立一個名稱為nginx-svc之service
    * 透過selector指向剛剛建立之pod(labels: `app: nginx version: "1"`)
    * port設定: 
        * local port: 80
        * service port: 12345
### 透過yaml建立service
```bash=
$ kubectl apply -f svc.yaml
service/nginx-svc created
```
檢查service及pod均正常運行
```bash=
$ kubectl get all
NAME             READY   STATUS    RESTARTS   AGE
pod/hands-on-2   2/2     Running   0          19m

NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/nginx-svc   ClusterIP   10.107.148.170   <none>        12345/TCP   18s
```
透過`port-forward`建立連線
```bash=
$ kubectl port-forward svc/nginx-svc 80:12345
Forwarding from 127.0.0.1:80 -> 80
Forwarding from [::1]:80 -> 80
Handling connection for 80
```
連上`127.0.0.1:80`檢查內容是否為pod中busybox echo之"Hands on 2 demo"
```bash=
$ curl 127.0.0.1:80
Hands on 2 demo
```