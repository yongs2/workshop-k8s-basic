# docker for mac 에서 시험

## dashboard 설치 

- [dashboard 설치](https://github.com/kubernetes/dashboard)
- deploy Dashboard

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

- proxy 기동 

```sh
kubectl proxy &
```

- 접근 계정 생성

```sh
kubectl get serviceaccount
kubectl create serviceaccount cluster-admin-dashboard-sa
kubectl create clusterrolebinding cluster-admin-dashboard-sa --clusterrole=cluster-admin --serviceaccount=default:cluster-admin-dashboard-sa
```

- dashboard 접근

```sh
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```
