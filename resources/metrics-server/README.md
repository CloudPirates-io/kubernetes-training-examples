### disable the metrics-server addon for minikube in case it was enabled, because it installs the metric-server@v0.2.1

```
$ minikube addons disable metrics-server
````

### now start a new minikube with token auth enabled
```
$ minikube delete; minikube start --extra-config=kubelet.authentication-token-webhook=true
````

### deploy the upgraded metric-server with insecure port and connection config
```
$ kubectl create -f deploy/
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.extensions/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
```