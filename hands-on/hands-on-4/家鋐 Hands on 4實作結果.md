###### tags: `K8S`
# 家鋐 Hands on 4實作結果
- 執行環境: Windows10 家用版
Docker Desktop 4.4.4 (73704) 
minikube version: v1.25.1
## :pencil: 實作一

詳見[家鋐 Homework week 2 實作結果](/-13A7lwITeGZwtfT_wMgeg)
 
## :pencil: 實作二
:dart: 以Helm Charts實現以上架構

#### 建立一新的chart
```bash=
$ >helm create hw
Creating hw
```

#### 目標helm資料夾結構
透過helm create建立chart後，將自動生成`Chart.yaml`, `values.yaml`及templates資料夾，現在仍缺少config及template資料夾中的yaml檔案

首先將mysql, nginx, wordpress這三個主要yaml檔放入templates資料夾中，並將configmap相關檔案開啟一新資料夾config做貯存。

```
│   .helmignore
│   Chart.yaml
│   values.yaml
│   
├───config
│       fluentd.conf
│       nginx-default.conf
│       
└───templates
        mysql.yaml
        nginx.yaml
        wordpress.yaml
```
- values.yaml檔案內容:
    - mysql.yaml之參數
    - nginx.yaml之參數
    - wordpress.yaml之參數


#### 改寫yaml檔內容
1. image: 方便更動版本
2. image pull policy: 控制鏡像下載
3. replicas: 方便調整
4. configmap內容統一移至config資料夾下方

#### deploy chart
1. 移至helm的資料夾位址
```bash=
$ cd helm
```
2. 執行install
```bash=
$ helm install wordpress wordpress
#執行結果
NAME: wordpress
LAST DEPLOYED: Wed Feb 16 11:02:14 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

3. 檢查是否各服務是否成功
```bash=
$ kubectl get all
#執行結果
NAME                             READY   STATUS    RESTARTS   AGE
pod/fluentd-cbmnt                1/1     Running   0          3m49s
pod/mysql-0                      1/1     Running   0          3m49s
pod/nginx-5c85958849-xt25h       1/1     Running   0          3m49s
pod/wordpress-65bd9cb878-r99z7   1/1     Running   0          3m49s

NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP          118m
service/mysql-svc       ClusterIP      None             <none>        3306/TCP         3m49s
service/nginx-svc       LoadBalancer   10.104.253.209   <pending>     8888:31442/TCP   3m49s
service/workpress-svc   ClusterIP      10.102.44.29     <none>        80/TCP           3m49s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/fluentd   1         1         1       1            1           <none>          3m49s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx       1/1     1            1           3m49s
deployment.apps/wordpress   1/1     1            1           3m49s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-5c85958849       1         1         1       3m49s
replicaset.apps/wordpress-65bd9cb878   1         1         1       3m49s

NAME                     READY   AGE
statefulset.apps/mysql   1/1     3m49s

NAME                                                REFERENCE              TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/nginx-hpa       Deployment/nginx       <unknown>/50%   1         3         0          3m49s
horizontalpodautoscaler.autoscaling/wordpress-hpa   Deployment/wordpress   <unknown>/30%   1         3         0          3m49s
```

4. 執行nginx服務
```bash=
$ minikube service nginx-svc
|-----------|-----------|-------------|---------------------------|
| NAMESPACE |   NAME    | TARGET PORT |            URL            |
|-----------|-----------|-------------|---------------------------|
| default   | nginx-svc |        8888 | http://192.168.49.2:31442 |
|-----------|-----------|-------------|---------------------------|
* Starting tunnel for service nginx-svc.
|-----------|-----------|-------------|------------------------|
| NAMESPACE |   NAME    | TARGET PORT |          URL           |
|-----------|-----------|-------------|------------------------|
| default   | nginx-svc |             | http://127.0.0.1:53186 |
|-----------|-----------|-------------|------------------------|
* Opening service default/nginx-svc in default browser...
! Because you are using a Docker driver on windows, the terminal needs to be open to run it.
```
