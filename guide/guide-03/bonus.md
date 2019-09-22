# Horizontal Pod Autoscaler

## [metric-server 설치 필요](https://github.com/kubernetes-incubator/metrics-server)

- 설치 확인

```sh
kubectl top node
kubectl top pod
```

- github 에서 checkout 이후 생성

```sh
# git clone https://github.com/kubernetes-incubator/metrics-server/
# pwd
/Users/yongs2/workspace/workshop-k8s-basic/guide/guide-03/metrics-server/deploy/1.8+
# kubectl create -f ./
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
# kubectl get all --all-namespaces
NAMESPACE       NAME                                            READY   STATUS    RESTARTS   AGE
docker          pod/compose-6c67d745f6-rft44                    1/1     Running   0          3d19h
docker          pod/compose-api-57ff65b8c7-tckx6                1/1     Running   0          3d19h
ingress-nginx   pod/nginx-ingress-controller-5b465c5589-t9q7z   1/1     Running   0          24h
kube-system     pod/coredns-584795fc57-2kks9                    1/1     Running   0          3d19h
kube-system     pod/coredns-584795fc57-l5cn6                    1/1     Running   0          3d19h
kube-system     pod/etcd-docker-desktop                         1/1     Running   0          3d19h
kube-system     pod/kube-apiserver-docker-desktop               1/1     Running   0          3d19h
kube-system     pod/kube-controller-manager-docker-desktop      1/1     Running   0          3d19h
kube-system     pod/kube-proxy-5mr47                            1/1     Running   0          3d19h
kube-system     pod/kube-scheduler-docker-desktop               1/1     Running   0          3d19h
kube-system     pod/kubernetes-dashboard-5f7b999d65-jc86w       1/1     Running   0          2d18h
kube-system     pod/metrics-server-844f95c776-2z6lr             1/1     Running   0          2m28s

NAMESPACE       NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
default         service/kubernetes             ClusterIP      10.96.0.1        <none>        443/TCP                      3d19h
docker          service/compose-api            ClusterIP      10.98.44.88      <none>        443/TCP                      3d19h
ingress-nginx   service/ingress-nginx          LoadBalancer   10.109.166.111   localhost     80:32481/TCP,443:30181/TCP   24h
kube-system     service/kube-dns               ClusterIP      10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       3d19h
kube-system     service/kubernetes-dashboard   ClusterIP      10.101.59.18     <none>        443/TCP                      2d18h
kube-system     service/metrics-server         ClusterIP      10.108.116.94    <none>        443/TCP                      2m28s

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   daemonset.apps/kube-proxy   1         1         1       1            1           <none>          3d19h

NAMESPACE       NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
docker          deployment.apps/compose                    1/1     1            1           3d19h
docker          deployment.apps/compose-api                1/1     1            1           3d19h
ingress-nginx   deployment.apps/nginx-ingress-controller   1/1     1            1           24h
kube-system     deployment.apps/coredns                    2/2     2            2           3d19h
kube-system     deployment.apps/kubernetes-dashboard       1/1     1            1           2d18h
kube-system     deployment.apps/metrics-server             1/1     1            1           2m28s

NAMESPACE       NAME                                                  DESIRED   CURRENT   READY   AGE
docker          replicaset.apps/compose-6c67d745f6                    1         1         1       3d19h
docker          replicaset.apps/compose-api-57ff65b8c7                1         1         1       3d19h
ingress-nginx   replicaset.apps/nginx-ingress-controller-5b465c5589   1         1         1       24h
kube-system     replicaset.apps/coredns-584795fc57                    2         2         2       3d19h
kube-system     replicaset.apps/kubernetes-dashboard-5f7b999d65       1         1         1       2d18h
kube-system     replicaset.apps/metrics-server-844f95c776             1         1         1       2m28s

# kubectl top node
error: metrics not available yet
# kubectl top pod
```

- metrics not avaliable yet 이라고 표시된다면, metrics-server 의 README 에 따라 다음과 같이 containers 의 image 항목 밑에 args 항목 3줄을 추가해야 함

```sh
# kubectl edit deploy -n kube-system metrics-server
...
    containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server-amd64:v0.3.4
        imagePullPolicy: Always
        args:
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
...
```

- 정상적으로 수정되었다면, 다음과 같이 출력되어야 함

```sh
# kubectl top node
NAME             CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
docker-desktop   502m         25%    1433Mi          75%
```

## 예제

guide-03/bonus/hpa-example-deploy.yml

```yml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: hpa-example-deploy
spec:
  selector:
    matchLabels:
      type: app
      service: hpa-example
  template:
    metadata:
      labels:
        type: app
        service: hpa-example
    spec:
      containers:
      - name: hpa-example
        image: k8s.gcr.io/hpa-example
        resources:
            limits:
              cpu: "0.5"
            requests:
              cpu: "0.25"
---

apiVersion: v1
kind: Service
metadata:
  name: hpa-example
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    type: app
    service: hpa-example
```


## 부하 테스트

### hpa-example 로 접속하는 클라이언트

```sh
kubectl run -it load-generator --image=busybox /bin/sh
while true; do wget -q -O- http://hpa-example; done # after 1 minute
```

- 위와 같이 runtime으로 실행시 부하 생성을 제대로 하기 위해서는 load-generator.yml 파일로 생성하여 테스트

### guide-03/bonus/hpa.yml

```yml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-example
spec:
  maxReplicas: 4
  minReplicas: 1
  scaleTargetRef:
    apiVersion: extensions/v1
    kind: Deployment
    name: hpa-example-deploy
  targetCPUUtilizationPercentage: 10
```

```
kubectl get hpa
```

- 다음과 같이 출력되고, 부하가 발생하면서, TARGETS 의 CPU 사용량 값이 증가하면, REPLICAS 가 증가하고, CPU 사용량이 감소하면, REPLICAS 감소함.

```sh
# kubectl get hpa
NAME          REFERENCE                       TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-example   Deployment/hpa-example-deploy   0%/10%    1         4         1          42m
```