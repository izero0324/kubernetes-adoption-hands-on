###### tags: `K8S`
# 家鋐 Homework week 1 實作結果
- 執行環境: Windows10 家用版
Docker Desktop 4.4.4 (73704) 
minikube version: v1.25.1

## 作業要求
1. 為 Pod YAML 加上 Resource Requests & Limits (因共用叢集，以下數值僅為讓各位練習使用，不等於生產環境之適合數值)
    * CPU requests (50m) & limits (100m)
    * Memory requests (50Mi) & limits (100Mi)
2. 為 Pod YAML 加上環境變數 `DUMMY_DATABASE_PASSWORD=admin`
3. 為 Pod YAML 加上一個 `busybox` container，並且
    * 利用 emptyDir 方式建立一個 shared volume 掛載進 busybox 與 nginx 容器內 (投影片 p53)
      * nginx mount path: `/usr/share/nginx/html/`
      * busybox mount path: `/html`
    * 利用 command 讓 busybox 持續 echo 剛剛新增的環境變數的值到 `/html/index.html` (投影片 p70)
4. 為 Pod YAML 加上三種探針 (投影片 p76~p86)
    * 檢測方式
        * Liveness、Startup 使用 `httpGet` 檢測 `/` 是否為 200
        * Readiness 使用 `tcpSocket` 檢測 `port 80`
    * 可參考 https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

## yaml撰寫想法
1. 首先先從新增container開始，在nginx旁再新增一個busybox的container，並加上使其執行對應指令
2. 接著將resource的request & limits還有環境變數加入。由於內含兩個container，因此均需分別於其中加入request及limit參數及環境變數。
3. 最後加入三種不同的探針，並將Liveness、Startup 設定為 httpGet模式檢測連線是否成功，並由tcp監測port80的readiness