# 윈도우용 k8s 관련 도구 다운로드 방법
### kubectl
- 다음 문서에서 Windows 설치 부분을 확인하여 다운로드함.
  * 이 예시에서는 1.30.7 버전을 설치함
  * https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-windows/#install-kubectl-binary-with-curl-on-windows
  * https://dl.k8s.io/release/v1.30.7/bin/windows/amd64/kubectl.exe

### eksctl
- 다음 경로에서 다운로드받을 것
https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_windows_amd64.zip

### ArgoCD CLI 
- 다음 주소에서 tagname으로 최신 버전 확인
  * https://api.github.com/repos/argoproj/argo-cd/releases/latest
- argocd cli 도구 파일 목록 확인
  * https://github.com/argoproj/argo-cd/releases/tag/v2.13.1
- 윈도우용 파일 다운로드
  * https://github.com/argoproj/argo-cd/releases/download/v2.13.1/argocd-windows-amd64.exe
- 다운로드한 파일을 적절한 디렉토리에 배치후 path 설정

### kubectl argo rollouts
- 다음의 경로에서 다운로드한 후 파일명을 kubectl-argo-rollouts.exe로 변경하여 사용
  * https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-windows-amd64

### helm 바이너리 다운로드 경로
- https://github.com/helm/helm/releases

