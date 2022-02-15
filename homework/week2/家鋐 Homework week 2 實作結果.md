###### tags: `K8S`
# 家鋐 Homework week 2 實作結果
- 執行環境: Windows10 家用版
Docker Desktop 4.4.4 (73704) 
minikube version: v1.25.1

:dart: 嘗試使用k8s物件設計一服務架構
#### 架構簡述
串接wordpress及MySQL，並由nginx控制，由fluentd紀錄

#### 服務架構設計想法
1. 從末端開始，首先透過==statefulset==建立`mysql`
2. 由於`mysql`需要使用者及密碼均屬於機敏資訊，因此透過`secret`的方式儲存
3. 建立`mysql-svc`的服務讀取`mysql`
4. 透過==deployment==建立一個`wordpress`的pod，並透過`secret`讀取連線`mysql`所需之資料
5. 建立`wordpress-svc`使`Nginx`得以存取其服務
6. 將`Nginx`及`Fluentd`建至於同個pod內使其可以輕易地透過volume共享資訊

### :hand:實作流程
1. 確認環境位於個人namespace中並資料夾位置為\hands-on\hands-on-4
2. 建立mysql的`secret`及`configMap`檔案:

    `mysql-secret` (base64轉換)
    - root_passwd: admin -> YWRtaW4=
    - user: blog_test -> YmxvZ190ZXN0
    - user_passwd: password -> cGFzc3dvcmQ=

    `mysql-cfgmap`
    
    - db_host: mysql-svc
    - db_name: Test_1234
並執行
```bash=
$ kubectl apply -f mysql_secret.yaml
secret/mysql-secret created # mysql的secret寫入成功
configmap/mysql-cfgmap created #mysql相關之configMap寫入成功
```

3. 透過==StatefulSet==建立`mysql`並建立`mysql-svc`
```bash=
$ kubectl apply -f mysql.yaml 
service/mysql-svc created #mysql svc
statefulset.apps/mysql created # mysql by statefulSet
```

4. 透過==Deployment==建立`wordpress`pod並產生其服務`wordpress-svc`
```bash=
$ kubectl apply -f wordpress.yaml
deployment.apps/wordpress created  # wordpress by deployment
service/wordpress-svc created # wordpress-svc
```
5. 檢查wordpress是否成功串接mysql並成功提供服務
```bash=
$ minikube service wordpress-svc -n esb22220
|-----------|---------------|--------------|---------------------------|
| NAMESPACE |     NAME      | TARGET PORT  |            URL            |
|-----------|---------------|--------------|---------------------------|
| esb22220  | wordpress-svc | wordpress/80 | http://192.168.49.2:31989 |
|-----------|---------------|--------------|---------------------------|
* Starting tunnel for service wordpress-svc.
|-----------|---------------|-------------|------------------------|
| NAMESPACE |     NAME      | TARGET PORT |          URL           |
|-----------|---------------|-------------|------------------------|
| esb22220  | wordpress-svc |             | http://127.0.0.1:63482 |
|-----------|---------------|-------------|------------------------|
* Opening service esb22220/wordpress-svc in default browser...
! Because you are using a Docker driver on windows, the terminal needs to be open to run it.
```
彈出wordpress初始設定視窗!
6. 透過==Deployment==建立`nginx`及`fluentd`，並建立`nginx-svc`
```bash=
$ kubectl apply -f nginx.yaml                                                                                                                 deployment.apps/nginx created
service/nginx-svc created
```