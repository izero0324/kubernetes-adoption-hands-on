###### tags: `K8S`
# 家鋐 Hands on 3實作結果
- 執行環境: Windows10 家用版
Docker Desktop 4.4.4 (73704) 
minikube version: v1.25.1
## :pencil: Deployment
:dart: 測試deployment controller:
- 加入readiness/liveness probe
- 設定resources的request/limit
- 測試滾動升級功能及回滾

### yaml撰寫:
- kind: Deployment
- 命名為hands-on-3-deployment
- 照投影片假定request及limit需求:
    - request: 
        - cpu: 50m 
        - memory: "50Mi"
    - limit:
        - cpu: 100m
        - memory: "100Mi"
- 加入probe: 觀測對象>nginx
    -  livenessProbe: 透過http port檢測回傳是否為200
    -  readinessProbe: 透過tcp檢查

### 透過yaml建立deployment
1. 於yaml檔目錄開啟cmd後，確認kubenetes正確執行並於指定之namespace下
```bash=
$ kubectl config view --minify | findstr namespace:
   namespace: esb22220
```
2. 透過yaml建立deployment
```bash=
$ kubectl apply -f deploy.yaml
deployment.apps/hands-on-3-deployment created
```
3. 檢查deployment是否成功建立及pod是否生成
```bash=
$ kubectl get all
NAME                                         READY   STATUS    RESTARTS   AGE
pod/hands-on-3-deployment-5fccc75b77-gkcpm   1/1     Running   0          7s

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hands-on-3-deployment   1/1     1            1           7s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/hands-on-3-deployment-5fccc75b77   1         1         1       7s
```
### 測試滾動升級
1. 更動yaml參數後重新deploy
2. 確認升級完成
```bash=
$ kubectl rollout status deployment hands-on-3-deployment
deployment "hands-on-3-deployment" successfully rolled out
```
3. 查看升級紀錄
```bash=
$ kubectl rollout history deployment hands-on-3-deployment
deployment.apps/hands-on-3-deployment
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```
4. 測試回滾至特定版本(第二次更新)
```bash=
$ kubectl rollout undo deployment hands-on-3-deployment --to-revision=2
deployment.apps/hands-on-3-deployment rolled back
```

## :pencil: DaemonSet
1. 作業要求: 為節點加上標籤`'network: 10g'`
```bash=
$kubectl label node minikube network=10g
node/minikube labeled
```
檢查標籤是否成功
```bash=
kubectl get nodes --show-labels
NAME       STATUS   ROLES                  AGE     VERSION   LABELS
minikube   Ready    control-plane,master   5h16m   v1.23.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=minikube,kubernetes.io/os=linux,minikube.k8s.io/commit=3e64b11ed75e56e4898ea85f96b2e4af0301f43d,minikube.k8s.io/name=minikube,minikube.k8s.io/updated_at=2022_02_14T09_27_22_0700,minikube.k8s.io/version=v1.25.1,network=10g,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=
```
在labels欄位找到`network=10g`之標籤
2. 將daemonset.yaml檔中加上設定使得其部屬到含有network:10g的節點上
做法: 由於目的是將container部屬至對應之node，因此在container的spec中加入nodeSelector選取指定之`network=10g`之標籤
3. 部屬daemonset
```bash=
$ kubectl apply -f daemonset.yaml
daemonset.apps/nginx-daemonset-1 created
```
檢查部屬結果
```bash=
kubectl get ds #ds=daemonset縮寫
NAME                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
nginx-daemonset-1   1         1         1       1            1           network=10g     16s
```
可以在node selector上看到已部署到選取之node上
4. 測試Rolling update及OnDelete更新策略之差異性

## :pencil: StatefulSet
:dart: 請修改 statefulset.yaml，滿足以下條件
1. 敏感資訊 (i.e 帳密)以 Secret 儲存
2. DB 連線資訊 以 Configmap 儲存

#### yaml檔撰寫想法
- 由於帳密資訊為較敏感之資訊，因此要求透過secret貯存，而將secret建立於同一yaml檔似乎並沒有達到分離敏感資訊的作用，我決定將secret及configmap獨立撰寫並獨立執行
- Secret之資訊需透過base64編碼
- Statefulset及deployment中用到之secret及configmap資訊透過ValueFrom的方式進行讀取

#### 測試結果
1. 設定Secret及configMap
```bash=
$ kubectl apply -f statefulset_secret.yaml
secret/mysql-secret created
configmap/mysql-cfgmap created
```
2. 部屬StatefulSet
```bash=
$ kubectl apply -f statefulset.yaml
service/mysql-svc created
statefulset.apps/mysql created
deployment.apps/myapp created
```
3. 檢查部屬結果
```bash=
$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/myapp-55548d7485-7lzhq   1/1     Running   0          35s
pod/mysql-0                  1/1     Running   0          35s

NAME                TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/mysql-svc   ClusterIP   None         <none>        3306/TCP   35s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp   1/1     1            1           35s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-55548d7485   1         1         1       35s

NAME                     READY   AGE
statefulset.apps/mysql   1/1     35s
```
4. 測試wordpress
```bash=
$ kubectl port-forward pod/myapp-55548d7485-7lzhq 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
```
5. 由於使用statefulset若刪除其中任意pod均會重新生成一個新的pod
```bash=
#刪除pod
$ kubectl delete pod mysql-0
pod "mysql-0" deleted
#透過另個terminal觀測pod之行為
$ kubectl get pods -w
NAME                     READY   STATUS    RESTARTS   AGE
myapp-55548d7485-7lzhq   1/1     Running   0          7m12s
mysql-0                  1/1     Running   0          7m12s
mysql-0                  1/1     Terminating   0          7m15s
mysql-0                  0/1     Terminating   0          7m18s
mysql-0                  0/1     Terminating   0          7m18s
mysql-0                  0/1     Terminating   0          7m18s
mysql-0                  0/1     Pending       0          0s
mysql-0                  0/1     Pending       0          0s
mysql-0                  0/1     ContainerCreating   0          0s
mysql-0                  1/1     Running             0          2s
```
可以看到在terminate mysql-0這個pod後，statefulset又重新開啟一個新的mysql-0的pod來維持運作

## :pencil: Job/CronJob

:dart: hands on目標
1. 查看 job / cronjob 執行結果
2. 暫時停止 cronjob: .spec.suspend 設為 true

#### job之yaml檔
與pod建立相同，將pod的內容放於job中之spec下方
cronJob與job相似，將job放置於其template內，並排程執行

#### 測試執行job及cronJob
執行job & cronJob
```bash=
$ kubectl apply -f job-cronjob.yaml
job.batch/hello created
cronjob.batch/hello created
```
由另一個terminal觀測job執行結果
```bash=
$ kubectl get pod -w
NAME          READY   STATUS    RESTARTS   AGE
hello-6xbsv   0/1     Pending   0          0s
hello-6xbsv   0/1     Pending   0          0s
hello-6xbsv   0/1     ContainerCreating   0          0s
hello-6xbsv   0/1     Completed           0          5s
hello-6xbsv   0/1     Completed           0          5s
hello-27413753-t6ft6   0/1     Pending             0          0s
hello-27413753-t6ft6   0/1     Pending             0          0s
hello-27413753-t6ft6   0/1     ContainerCreating   0          0s
hello-27413753-t6ft6   0/1     Completed           0          5s
hello-27413753-t6ft6   0/1     Completed           0          5s
hello-27413754-2mz5m   0/1     Pending             0          0s
hello-27413754-2mz5m   0/1     Pending             0          0s
hello-27413754-2mz5m   0/1     ContainerCreating   0          0s
hello-27413754-2mz5m   0/1     Completed           0          4s
hello-27413754-2mz5m   0/1     Completed           0          4s
hello-27413753-t6ft6   0/1     Terminating         0          64s
hello-27413753-t6ft6   0/1     Terminating         0          64s
```
可以看到依序執行了job後 等待cronJob排程執行條件滿足(每分鐘執行一次)後執行

#### 暫時停止 cronjob
1. 查看cronjob
```bash=
$ kubectl get cronJob
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        48s             5m46s
```
2. 透過`edit`將suspend設定為true
```bash=
$ kubectl get cronJob
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   True      0        51s             7m49s
```
持續透過get pod觀察發現接下來均無產生新的pod