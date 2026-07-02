# 쿠버네티스 · AKS 명령어 & 트러블슈팅 치트시트

> az = 클러스터 바깥(인프라) 관리 · kubectl = 클러스터 안(워크로드) 관리

---

## 1. Azure CLI (az) — 인프라 관리

### 로그인 / 기본
| 명령어 | 용도 |
|---|---|
| `az login` | Azure 로그인 |
| `az account show` | 현재 구독 정보 확인 |
| `az account show --query id -o tsv` | 구독 ID만 뽑기 |

### 리소스 그룹 (리소스를 묶는 폴더)
| 명령어 | 용도 |
|---|---|
| `az group create --name <이름> --location koreacentral` | 생성 |
| `az group list -o table` | 목록 확인 |
| `az group show --name <이름>` | 상세 확인 |
| `az group delete --name <이름> --yes --no-wait` | 삭제 (안에 든 것 전부) |

### AKS 클러스터
| 명령어 | 용도 |
|---|---|
| `az aks create -g <그룹> -n <클러스터> --node-count 1` | 생성 |
| `az aks list -o table` | 목록 확인 |
| `az aks show -g <그룹> -n <클러스터>` | 상세 확인 |
| `az aks get-credentials -g <그룹> -n <클러스터>` | kubectl 연결 설정 |
| `az aks update -g <그룹> -n <클러스터> --attach-acr <ACR>` | ACR 연결 |
| `az aks scale -g <그룹> -n <클러스터> --node-count 3` | 노드 수 조절 |
| `az aks delete -g <그룹> -n <클러스터> --yes` | 클러스터만 삭제 |

### ACR (컨테이너 이미지 저장소)
| 명령어 | 용도 |
|---|---|
| `az acr create -g <그룹> -n <ACR이름> --sku Basic` | 생성 |
| `az acr list -o table` | 목록 확인 |
| `az acr build --registry <ACR> --image <이름>:<태그> .` | 이미지 빌드 + 저장 |
| `az acr repository list -n <ACR>` | 저장된 이미지 목록 |

### 서비스 등록
| 명령어 | 용도 |
|---|---|
| `az provider register --namespace Microsoft.ContainerRegistry` | 서비스 활성화 |
| `az provider show --namespace Microsoft.ContainerRegistry --query registrationState -o tsv` | 등록 상태 확인 |

---

## 2. kubectl — 워크로드 관리

### 확인 (get / describe) — 가장 많이 사용
| 명령어 | 용도 |
|---|---|
| `kubectl get nodes` | 노드 목록·상태 |
| `kubectl get pods` | Pod 목록 (default 네임스페이스) |
| `kubectl get pods -A` | 전체 네임스페이스 Pod |
| `kubectl get pods -o wide` | 더 자세히 (어느 노드에 떴는지 등) |
| `kubectl get service` | Service 목록 (외부 IP 확인) |
| `kubectl get deployment` | Deployment 목록 |
| `kubectl get all` | 주요 리소스 한 번에 |
| `kubectl describe pod <이름>` | 특정 Pod 상세·이벤트 (문제 원인 파악) |

> `get` = 목록·상태 훑기 / `describe` = 하나를 깊게 파보기. Pod가 안 뜰 때는 `describe` 하단 이벤트를 확인.

### 생성 · 적용 (apply)
| 명령어 | 용도 |
|---|---|
| `kubectl apply -f <파일>.yaml` | YAML 적용 (생성 또는 갱신) ★ 가장 많이 씀 |
| `kubectl apply -f <폴더>/` | 폴더 안 YAML 전부 적용 |

> `apply`는 "이 파일 내용대로 맞춰줘". 새로 만들 때도, 고칠 때도 같은 명령.

### 수정
| 명령어 | 용도 |
|---|---|
| `kubectl scale deployment <이름> --replicas=5` | Pod 개수 조절 |
| `kubectl set image deployment/<이름> <컨테이너>=<새이미지>` | 이미지 교체 (배포) |
| `kubectl rollout restart deployment <이름>` | Pod 강제 재시작 (교체) |
| `kubectl rollout status deployment <이름>` | 배포 진행 상황 확인 |
| `kubectl edit deployment <이름>` | 리소스 직접 열어서 수정 |

### 삭제 (delete)
| 명령어 | 용도 |
|---|---|
| `kubectl delete pod <이름>` | Pod 삭제 (Deployment가 다시 만듦 = 자기복구) |
| `kubectl delete -f <파일>.yaml` | 그 파일에 정의된 것 전부 삭제 |
| `kubectl delete deployment <이름>` | Deployment 삭제 |
| `kubectl delete service <이름>` | Service 삭제 (외부 IP 회수) |

### 디버깅 (문제 생겼을 때)
| 명령어 | 용도 |
|---|---|
| `kubectl logs <Pod이름>` | Pod 로그 보기 |
| `kubectl logs -f <Pod이름>` | 로그 실시간 따라가기 |
| `kubectl exec -it <Pod이름> -- /bin/bash` | Pod 안으로 들어가기 |
| `kubectl get events` | 클러스터 이벤트 (뭐가 잘못됐는지) |

---

## 3. 자주 쓰는 공통 옵션

| 옵션 | 용도 |
|---|---|
| `-n <네임스페이스>` | 특정 네임스페이스 대상 (예: `-n kube-system`) |
| `-A` / `--all-namespaces` | 전체 네임스페이스 |
| `-o wide` / `-o yaml` / `-o json` | 출력 형식 |
| `--watch` / `-w` | 실시간 감시 (Ctrl+C로 종료) |
| `-f <파일>` | 파일 지정 |

---

## 4. 자주 만나는 에러 & 해결법 (트러블슈팅)

### 에러 1: 자격증명 만료 (가장 흔함)

**증상**
```
error: You must be logged in to the server (the server has asked for the client to provide credentials)
```
또는
```
error: error validating "nginx.yaml": ... the server has asked for the client to provide credentials
```

**원인**: kubectl 인증 토큰 만료. 시간이 지나거나 Cloud Shell 세션이 새로 열리면 자연스럽게 발생 (잘못한 것 아님).

**해결**: 자격증명 다시 받기
```
az aks get-credentials \
  --resource-group K8s_test \
  --name web-test \
  --overwrite-existing
```
그래도 안 되면 (Azure 로그인 자체가 풀린 경우):
```
az login
# 그다음 위 get-credentials 다시 실행
```

---

### 에러 2: 서비스가 구독에 등록 안 됨

**증상**
```
(MissingSubscriptionRegistration) The subscription is not registered to use namespace 'Microsoft.ContainerRegistry'.
```

**원인**: 해당 Azure 서비스(예: ACR)를 구독에서 처음 써서 활성화가 안 됨.

**해결**: 서비스 등록 후 Registered 확인
```
az provider register --namespace Microsoft.ContainerRegistry
# 1~2분 뒤 상태 확인 (Registered 나올 때까지)
az provider show --namespace Microsoft.ContainerRegistry --query registrationState -o tsv
# Registered 확인되면 원래 하려던 명령 다시 실행
```

---

### 에러 3: 꺾쇠 괄호를 그대로 입력

**증상**
```
bash: K8s_test: No such file or directory
```

**원인**: `<K8s_test>` 처럼 `< >`를 실제로 입력함. 괄호는 "여기에 값을 넣으라"는 표시일 뿐이고, bash에서 `<`는 파일 입력 기호로 해석됨.

**해결**: 괄호 빼고 이름만 입력
```
# 잘못된 예
az aks get-credentials --resource-group <K8s_test> --name <web-test>
# 올바른 예
az aks get-credentials --resource-group K8s_test --name web-test
```

---

### 에러 4: 웹 페이지가 nginx 기본 페이지로 나옴

**증상**: 내 HTML로 바꿨는데 "Welcome to nginx!" 기본 페이지가 뜸.

**원인**: Dockerfile의 COPY 도착지가 `index.html`이 아니거나, 파일명만 바꾸고 Dockerfile을 안 고침. nginx는 기본으로 `index.html`을 찾음.

**해결**: COPY 오른쪽(도착지)을 반드시 `index.html`로
```
FROM nginx:latest
# 왼쪽(내 파일명)은 자유, 오른쪽(nginx가 읽는 위치)은 index.html 고정
COPY index1.html /usr/share/nginx/html/index.html
```
Actions 탭에서 워크플로가 빨간 X면 빌드 실패 → 로그 확인.

---

### 에러 5: EXTERNAL-IP가 계속 pending

**증상**: `kubectl get service`에서 EXTERNAL-IP가 `<pending>` 상태.

**원인**: Azure가 로드밸런서와 공개 IP를 만드는 중. 보통 1~2분 걸림.

**해결**: 잠시 기다리며 감시
```
kubectl get service <서비스이름> --watch
# IP가 뜨면 Ctrl+C로 종료
```
2~3분 넘게 계속 pending이면 `kubectl describe service <이름>`로 이벤트 확인.

---

## 5. 전체 흐름 (구축 → 운영 → 정리)

1. **인프라 준비** — `az group create` → `az aks create` → `az aks get-credentials`
2. **이미지 저장소 연결** — `az acr create` → `az aks update --attach-acr`
3. **빌드 & 배포** — `az acr build` → `kubectl apply -f`
4. **확인** — `kubectl get pods` / `kubectl get service`
5. **운영 중 조절** — `kubectl scale` / `kubectl set image`
6. **정리 (과금 방지)** — `az group delete`

---

## 6. 기억할 감각

- **az = 클러스터 바깥(인프라), kubectl = 클러스터 안(워크로드)**
- **확인은 `get`(훑기)과 `describe`(파보기)** — 문제 생기면 `describe`와 `logs`
- **생성·수정은 거의 다 `apply -f` 하나로 통일** (YAML 고치고 다시 apply)
- **삭제는 `delete`**, 전체 정리는 `az group delete`
- **credentials 에러는 `az aks get-credentials`로 재발급** (자주 겪는 정상 상황)
- **실습 끝나면 반드시 리소스 삭제** — AKS는 켜둔 시간만큼 과금됨

---

*본인 환경 값: 리소스그룹 `K8s_test` · 클러스터 `web-test` · ACR `wleks55555acr`*
