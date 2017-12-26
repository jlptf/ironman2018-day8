## Day 8 - 與 k8s 溝通: kubectl

### 本日共賞

* kubectl
* kubectl 常用指令

### 希望你知道

* [安裝 k8s](https://ithelp.ithome.com.tw/articles/10192748)
* [運行 minikube](https://ithelp.ithome.com.tw/articles/10193237)

[與 k8s 溝通: APIs](https://ithelp.ithome.com.tw/articles/10193489) 與 [與 k8s 溝通: Dashboard](https://ithelp.ithome.com.tw/articles/10193497) 分別介紹兩種與 k8s 溝通的方式，今天要介紹第三種方式：kubectl

#### kubectl

kubectl 是一支用來與 k8s 叢集溝通的二進位 (binary) 工具。我們可以運用 kubectl 來

* 取得 k8s 各種不同資源資訊 (get: pod, service, ingress, ...)
* 取得 k8s 各種不同資源的詳細內容 (describe: pod, service, ingress, ...)
* 配置 k8s 運行資源 (create, apply, rollout, ...)
* 刪除 k8s 運行資源 (delete)
* 取得 log 檔案 (logs)

#### kubectl 常用指令

**1. 取得設定檔**

```bash
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/james/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/james/.minikube/client.crt
    client-key: /Users/james/.minikube/client.key
```

這裡會列出目前 kubectl 的設定內容。設定檔會放置在 ~/.kube/config (mac or linux) ，上述內容表示 kubectl 已連接到 minikube。

**2. 管理多個 k8s 叢集**

```bash
$ kubectl config use-context [NAME]
```

如果你需要管理多個 k8s 叢集，可以利用上面指令作切換，例如切換到 `minikube` 

```bash
$ kubectl config use-context minikube
Switched to context "minikube".
```

如果想查詢有哪些叢集可以切換：

```bash
$ kubectl config get-contexts
CURRENT   NAME          CLUSTER          AUTHINFO          NAMESPACE  
*         minikube      minikube         minikube  
```

> 如果你只有一個叢集能管理，當然只會列出一個

如果想查看目前正在管理的叢集

```bash
$ kubectl config current-context
minikube
```


**3. 取得叢集狀態**

```bash
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

這裡會列出 k8s 叢集的運行資訊。根據上面資訊可得知 master 正運行在 https://192.168.99.100:8443

>還記得 Master Node 嗎？


**4. 代理**


```bash
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

需要與 k8s 溝通時，必須要先經由 api server 認證後才可以存取 k8s 資源。透過 kubectl proxy，kubectl 會與 api server 進行認證溝通。完成認證之後，就可以利用瀏覽器開啟網址 localhost:8001/ui 使用 Dashboard 或透過 API 進行操作。如果想查詢有哪些 APIs 可使用，可透過 curl 指令來查詢。

```bash
$ curl http://localhost:8001
{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1alpha1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1beta1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1beta1",
    "/apis/apps",
...

```

> 請注意，查詢 APIs 必須要先執行 kubectl proxy 通過認證後才可執行

**5. 建立命名空間**

```bash
$ kubectl create namespace [name]
```

在 k8s 中，命名空間 (namespace) 是很重要的觀念。利用命名空間，你可以隔開兩個不同的群組或環境，例如 `deveop` 表示開發環境 `production` 表示正是環境。而在不同環境內的物件並不會互相干擾，也可以有相同的名稱。當 k8s 被建立的同時，會有三個命名空間產生，分別是 `default`, `kube-public` 與 `kube-system`。看字面應該也很容易知道 `kube-public` 與 `kube-system` 是 k8s 內部使用的。而 `default` 則是預設的命名空間，即在未指定部署空間時，所有的物件都會被部署在 `default` 內。如果你想要建立只要透過上面格式就可輕鬆建立。例如：建立一個 `develop`

```
$ kubectl create namespace develop
namespace "develop" created
```

**6. 取得資源**

```bash
$ kubectl get [resources][namespace]
```

在 k8s 中有各式各樣不同的資源物件。例如 Pod, Secret, Service, Namespace 等等，我們可以利用上述指令來查詢叢集中現有的資源。

> 不知道什麼是 Pod, Service, .. 嗎？別擔心！之後的文章會再詳細說明。目前先專注在 kubectl 指令上即可

*範例一 查詢命名空間 (namespace)*

```bash
$ kubectl get namespace
NAME          STATUS    AGE
default       Active    4d
kube-public   Active    4d
kube-system   Active    4d
```

這裡列出目前叢集內正在運行的三個命名空間，分別是 `default`, `kube-public` 與 `kube-system`

> 請注意，這個範例並沒有指定 [namespace] 而是查詢 `namespace`。

*範例二 查詢在 `develop` 命名空間 (namespace) 中運行的 pod*

```bash
kubectl get pods --namespace=develop
NAME                             READY     STATUS    RESTARTS   AGE
webserver-2736313971-t86gs       1/1       Running   2          29d
webserver-red-4210812574-tdlrz   1/1       Running   2          29d
```

這裡列出在 `develop` 這個命名空間內

* 有兩個 pod 正在運行 (running)
* 重啟次數 (RESTARTS：2) 為 2 次
* 已經運行了 29 天 (AGE：29d)
* 而 pod 名稱 (webserver-2736313971-t86gs) 在隨後的取得資源詳細內容的說明時會使用到。

> * 如果未指定命名空間 (namespace)，則預設會查詢 default 這個命名空間。
>
> * 在未進行任何部署之前，只有預設的 `kube-system` 與 `kube-public` 這兩個命名空間能查到 Pod 資源。
>
> * 上述的例子中，已經有兩個 Pod 被部署在 develop 的命名空間內，故 get pods 指令才能查詢到資料，至於如何部署 Pod ，我們會在之後的文章再詳細說明。目前僅就 kubectl 常用指令進行說明。你可以嘗試看看把 [namespace] 修改成 `kube-system` 看看能看到什麼！

*範例三 查詢在 production 命名空間 (namespace) 中運行的 service*

```bash
$ kubectl get services --namespace=production
NAME            CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
kubernetes      10.0.0.1     <none>        443/TCP        33d
webserver-svc   10.0.0.35    <nodes>       80:31771/TCP   29d
```

這裡列出在 `production` 命名空間內

* 共有兩個 service 正在運行以及被分配的 ip 分別為 10.0.0.1 及 10.0.0.35
* 其中一個 service (kubernetes)
  * 並未開放任何外部存取 ip (EXTERNAL-IP：<none>) 
  * 佔用 443 port。
* 另外一個 service (webserver-svc)
  * 可透過實體主機 (EXTERNAL-IP：<nodes>) 經由 tcp 31771 port 來存取。
  * 80:31771/TCP 指的是 pod 中的 80 port 會對應到實體主機的 31771 port

> 可執行 kubectl [action] --help 查看說明，例如 kubectl get --help
> 
> 如何部署 Service ，我們會在之後的文章再詳細說明。


**7. 取得資源詳細內容**

```bash
$ kubectl describe [resources][namespace]
```

若想獲得更詳細的資源內容，我們可以利用上述指令來取得。根據上述 get 指令我們可以知道目前 develop 內共有兩個 pod 正在運行，如果想獲得更詳細的 pod 資訊，可使用下列指令

```bash
$ kubectl describe pods webserver-2736313971-t86gs --namespace=develop
```

> 物件每次新建或重啟都會被賦予不同的名字，請記得將 "webserver-2736313971-t86gs" 替換成正確的名稱。另外你可以試著查查看運行在 `kube-system` 的 Pod

**8. 刪除資源**

```bash
$ kubectl delete [resources][namespace]
```

若要刪除叢集內的資源可透過上述指令進行刪除

*範例一 刪除 `develop` 命名空間內名為 webserver-2736313971-t86gs 的 pod*

```bash
$ kubectl delete pods webserver-2736313971-t86gs --namespace=develop
pod "webserver-2736313971-t86gs" deleted
```

> 你應該不會想要刪除 `kube-system` 裡面的 Pod 吧？為什麼不！試試看吧！你會發現 k8s 會再幫你重新建立喔！

*範例二 刪除 `production` 命名空間內名為 webserver-svc 的 service*

```bash
$ kubectl delete services webserver-svc --namespace=production
service "webserver-svc" deleted
```

**9. 部署**

```bash
$ kubectl apply -f [resources file/folder][namespace]
```

如果有 k8s .yaml 描述檔案，我們可透過 apply 部署 k8s 資源

> 除了利用 apply 部署，還可以利用 run 或 expose 等等指令來建立物件。使用指令雖然快速方便，但如果需要同時部署多個物件會比較不方便。另外，使用 yaml 檔也可以有個紀錄，方便追蹤除錯。因此，之後的說明，會使用 yaml 為主。
> 
> 底下是一個利用 run 來部署一個 nginx 的範例
>
>```bash
>$ kubectl run nginx --image=nginx
>deployment "nginx" created
>
>$ kubectl get deployment
>NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
>nginx     1         1         1            1           4m
>```


更完整的 kubectl 指令說明與資源類別，請參考 [kubectl 官方指令說明](https://kubernetes.io/docs/user-guide/kubectl-overview/)

本文同步發表於 [https://jlptf.github.io/ironman2018-day8/](https://jlptf.github.io/ironman2018-day8/)
