

# 用yaml安装 k8s-dashboard

## 来源

https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard/



## 安装

执行

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

得到

```shell
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard configured
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard configured
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```



## 隧道映射到本地端口

执行

```shell
kubectl get all -n kubernetes-dashboard
```

得到

```shell
NAME                                            READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-c79c65bb7-l44kk   1/1     Running   0          2m40s
pod/kubernetes-dashboard-56484d4c5-wc2cf        1/1     Running   0          2m43s

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   172.21.244.68    <none>        8000/TCP   2m45s
service/kubernetes-dashboard        ClusterIP   172.21.204.148   <none>        443/TCP    2m58s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           2m44s
deployment.apps/kubernetes-dashboard        1/1     1            1           2m47s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-c79c65bb7   1         1         1       2m44s
replicaset.apps/kubernetes-dashboard-56484d4c5        1         1         1       2m47s

```

> ⬆️注意  pod/kubernetes-dashboard-56484d4c5-wc2cf 

> 注意  如果pod一直都是creating的状态，可以用👇的方法观察等待

```shell
kubectl get pod -n kubernetes-dashboard -w
```

执行

```shell
export POD_NAME=kubernetes-dashboard-56484d4c5-wc2cf
kubectl -n kubernetes-dashboard port-forward $POD_NAME 8443:8443
```

打开浏览器 https://127.0.0.1:8443/

![](https://raw.githubusercontent.com/zknyy/k8s-dashboard-install/master/login.png)

## 获取Bearer Token

执行

```shell
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

得到

> 此处有可能得到不止一种Token，但只需要  Namespace:    kubernetes-dashboard

```
Name:         kubernetes-dashboard-token-j8kbc
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard
              kubernetes.io/service-account.uid: 4ab193c8-40dd-4a76-b8f7-5c9df8a2afdb

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Ijc5RVJpbUl0RzdYbFZpVHAyejQ2QThnS2stczVtRUFmTTcwSC1QNzUzLTQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi1qOGtiYyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjRhYjE5M2M4LTQwZGQtNGE3Ni1iOGY3LTVjOWRmOGEyYWZkYiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.VKwdRclWDu6CvPKxl_Sjzimea_GdXwhuHOWbTQ49W0SdcbJ5sRwYs1xhfCdP0Ldv2a9uygkU63YaY_EIDwyQ0aDaF1EZ-MIb5feSgkWGV8AEAX2C6Ctd3__e-H7vv_y_B69ykVP0Wj2viWC9zVzDBUhTl7Plro_xP4QJZKvdyF9xzpOo-TjW41Fa_lf3BG9XJctHbhMMWR1Kcm1VjFFAvEJWH1__g9tqEY65QFCYUUyaLYlPo2qyo7jZuMtlA9mpix1YJJjnYnRg_6POP609N5aYPdTmket6ZVokrH4BWlF0xM6OFlWHgp-nNWJ5ij2ZInKT9GC3wwHOCup5xPdOmQN9MP2wvCWVO-iIhbhdzKKYfduBEEbIQ5L0_Ku2Mmia3yDw6wT7L___B1RBtrg4OYFRTwJe1nNY848ln1Uxthdw6YVHFlijJ3fnKpvNGPsR6ww70CdyNhYMzC7w6ZODflTm3fcVewewYJVHIvBOPInI4ILdHnYFqjk0Jn52bhmxLaOiZZIXisEYzayx8xv2A5bJajFr-h0MJn7z9dYpil7ql1FlsbGDqQXLL0A9_TsyKLQKnBD0NRR9aiBmEanffg0XpZbNm8bCD3PbltwyRZ-mguJ4H8gEbDQp6bHuM17dxLIUj6_GtffyMH0NF9pjQYlg7N9shm5wT404BuDz0p0
ca.crt:     1874 bytes
namespace:  20 bytes
```

选第一项，把token拷贝出来，直接粘贴到输入框中

![](https://raw.githubusercontent.com/zknyy/k8s-dashboard-install/master/fill-token.png)

然后sign in

![](https://raw.githubusercontent.com/zknyy/k8s-dashboard-install/master/dashboard.png)



=================================================================以下没有完成=================================================

# 用Helm安装 k8s-dashboard

## 来源

https://hub.helm.sh/charts/k8s-dashboard/kubernetes-dashboard



## 安装

 执行

```shell
helm repo add k8s-dashboard https://kubernetes.github.io/dashboard
helm install k8s-dashboard/kubernetes-dashboard --version 2.0.1 --generate-name
```

得到

```shell
NAME: kubernetes-dashboard-1592309946
LAST DEPLOYED: Tue Jun 16 20:19:10 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
*********************************************************************************
*** PLEASE BE PATIENT: kubernetes-dashboard may take a few minutes to install ***
*********************************************************************************

Get the Kubernetes Dashboard URL by running:
  export POD_NAME=$(kubectl get pods -n default -l "app.kubernetes.io/name=kubernetes-dashboard,app.kubernetes.io/instance=kubernetes-dashboard-1592309946" -o jsonpath="{.items[0].metadata.name}")
  echo https://127.0.0.1:8443/
  kubectl -n default port-forward $POD_NAME 8443:8443
```

## 隧道映射到本地端口

执行

```shell
export POD_NAME=$(kubectl get pods -n default -l "app.kubernetes.io/name=kubernetes-dashboard,app.kubernetes.io/instance=kubernetes-dashboard-1592309946" -o jsonpath="{.items[0].metadata.name}")
echo https://127.0.0.1:8443/
kubectl -n default port-forward $POD_NAME 8443:8443
```

## 准备登陆

浏览器打开 https://127.0.0.1:8443/

![](https://raw.githubusercontent.com/zknyy/k8s-dashboard-install/master/login.png)

## 授权

从K8s获取权限，并通过SA绑定到Dashboard

### 创建SA文件

创建一个 `dashboard-adminuser.yaml` 文件

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

### 创建admin-user资源

执行

```shell
kubectl apply -f dashboard-adminuser.yaml
```







## 其他参考

https://github.com/kubernetes/dashboard

https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard/