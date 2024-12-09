# k8s에 argocd 설치

## 사전 설치
- vagrant + VitualBox 기반으로 k8s를 설치합니다.
  * [이 문서를 이용하여 설치합니다.](https://github.com/stepanowon/k8s-multi-node) 
- k8s와 metalLB 까지의 설정을 완료합니다. 

## ArgoCD 설치
```sh
# 설치
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# argocd-server 서비스의 Type을 LoadBalancer로 변경
### 리눅스
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

### 윈도우
kubectl patch svc argocd-server -n argocd -p '{\"spec\": {\"type\": \"LoadBalancer\"}}'
```

```sh
# 설치 확인 : service/argocd-server 서비스의 EXTERNAL-IP가 argocd-server 엔드포인트임.
# 이 예시에서의 엔드포인트는 192.168.56.51
$ kubectl get all -n argocd
NAME                                                    READY   STATUS    RESTARTS      AGE
pod/argocd-application-controller-0                     1/1     Running   0             116s
pod/argocd-applicationset-controller-6899bbc884-9v5nx   1/1     Running   0             118s
pod/argocd-dex-server-bcf886644-fsq24                   1/1     Running   1 (47s ago)   117s
pod/argocd-notifications-controller-64fb4bc4f9-pjd9k    1/1     Running   0             117s
pod/argocd-redis-6cbf9bf4c5-cxjk4                       1/1     Running   0             117s
pod/argocd-repo-server-b5d548b6c-w5wkr                  1/1     Running   0             117s
pod/argocd-server-65bc9998bd-98cd2                      1/1     Running   0             116s

NAME                                              TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP      10.105.11.206    <none>          7000/TCP,8080/TCP            119s
service/argocd-dex-server                         ClusterIP      10.107.210.139   <none>          5556/TCP,5557/TCP,5558/TCP   119s
service/argocd-metrics                            ClusterIP      10.106.12.100    <none>          8082/TCP                     118s
service/argocd-notifications-controller-metrics   ClusterIP      10.97.74.102     <none>          9001/TCP                     118s
service/argocd-redis                              ClusterIP      10.97.207.215    <none>          6379/TCP                     118s
service/argocd-repo-server                        ClusterIP      10.101.45.67     <none>          8081/TCP,8084/TCP            118s
service/argocd-server                             LoadBalancer   10.100.74.201    192.168.56.51   80:30363/TCP,443:31381/TCP   118s
service/argocd-server-metrics                     ClusterIP      10.96.89.242     <none>          8083/TCP                     118s

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   1/1     1            1           118s
deployment.apps/argocd-dex-server                  1/1     1            1           118s
deployment.apps/argocd-notifications-controller    1/1     1            1           118s
deployment.apps/argocd-redis                       1/1     1            1           117s
deployment.apps/argocd-repo-server                 1/1     1            1           117s
deployment.apps/argocd-server                      1/1     1            1           117s

NAME                                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-6899bbc884   1         1         1       118s
replicaset.apps/argocd-dex-server-bcf886644                   1         1         1       118s
replicaset.apps/argocd-notifications-controller-64fb4bc4f9    1         1         1       117s
replicaset.apps/argocd-redis-6cbf9bf4c5                       1         1         1       117s
replicaset.apps/argocd-repo-server-b5d548b6c                  1         1         1       117s
replicaset.apps/argocd-server-65bc9998bd                      1         1         1       116s

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     116s
```

## ArgoCD CLI 도구 설치
#### Linux 설치
```sh
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```
#### Windows 설치
- 이 [문서](windows-tool.md)를 참조하여 설치합니다.

## ArgoCD 초기 패스워드 확인
```sh
$ argocd admin initial-password -n argocd
9Sn4IHy68plepGak

 This password must be only used for first time login. We strongly recommend you update the password using `argocd account update-password`.
```

## ArgoCD 서버 접속
- 브라우저를 열고 argocd-server 서비스의 EXTERNAL-IP로 접속
  * 이 예시에서의 EXTERNAL-IP : 192.168.56.51
- 인증서 관련 안전하지 않다는 경고가 나타나면 '고급' 링크를  '계속하기'를 클릭함
- admin 사용자 + 초기 패스워드로 로그인함
- 로그인 후 왼쪽 메뉴에서 'User Info'를 클릭한 뒤 admin 계정의 패스워드를 변경함

## ArgoCD CLI를 이용한 로그인
```sh
# argocd-server 서비스의 EXTERNAL-IP로 접속함
$ argocd login 192.168.56.51
WARNING: server certificate had error: tls: failed to verify certificate: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Username: admin
Password:
'admin:login' logged in successfully
Context '192.168.56.51' updated
```

## 새로운 사용자 추가
```sh
# argocd-cm configmap 에 새로운 사용자 추가
$ kubectl edit configmap argocd-cm -n argocd 

# 다음 내용을 추가(user1 사용자 추가)
data: 
  accounts.user1: apiKey, login

# 새로운 사용자의 패스워드는 관리자의 것을 상속받음
```

## 새로운 사용자의 패스워드 변경
```sh
# user1 계정의 패스워드를 user1pwd로 설정 
# Linux
argocd account update-password  \
  --account user1  \
  --current-password adminpwd  \
  --new-password user1pwd

# 윈도우 
argocd account update-password  `
  --account user1  `
  --current-password adminpwd  `
  --new-password user1pwd

```

## 새로운 사용자에 대한 권한 설정
```sh
$ kubectl edit configmap argocd-rbac-cm -n argocd 

# 다음 내용을 추가함. user1에게 관리자 권한 부여
data:
  policy.csv: |
    g, user1, role:admin
  policy.default: role.readonly

```

## ArgoCD CLI, 브라우저 화면에서 user1 계정으로 로그인
```sh
$ argocd logout 192.168.56.51
$ argocd login 192.168.56.51
```

