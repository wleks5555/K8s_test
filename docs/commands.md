# 쿠버네티스 · AKS 명령어 치트시트

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

### 서비스 등록 (MissingSubscriptionRegistration 에러 해결)
| 명령어 | 용도 |
|---|---|
| `az provider register --namespace Microsoft.ContainerRegistry` | 서비스 활성화 |
| `az provider show --namespace Microsoft.ContainerRegistry --query registrationState -o tsv` | 등록 상태 확인 (Registered 확인) |

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
| `-o wide` / `-o yaml` / `-o json` | 출력 형식 (wide=자세히, yaml/json=전체 정의) |
| `--watch` / `-w` | 실시간 감시 (Ctrl+C로 종료) |
| `-f <파일>` | 파일 지정 |

---

## 4. 전체 흐름 (구축 → 운영 → 정리)

1. **인프라 준비**
   `az group create` → `az aks create` → `az aks get-credentials`
2. **이미지 저장소 연결**
   `az acr create` → `az aks update --attach-acr`
3. **빌드 & 배포**
   `az acr build` → `kubectl apply -f`
4. **확인**
   `kubectl get pods` / `kubectl get service`
5. **운영 중 조절**
   `kubectl scale` / `kubectl set image`
6. **정리 (과금 방지)**
   `az group delete`

---

## 5. 기억할 감각

- **az = 클러스터 바깥(인프라), kubectl = 클러스터 안(워크로드)**
- **확인은 `get`(훑기)과 `describe`(파보기)** — 문제 생기면 `describe`와 `logs`
- **생성·수정은 거의 다 `apply -f` 하나로 통일** (YAML 고치고 다시 apply)
- **삭제는 `delete`**, 전체 정리는 `az group delete`
- **실습 끝나면 반드시 리소스 삭제** — AKS는 켜둔 시간만큼 과금됨

---

*본인 환경 값: 리소스그룹 `K8s_test` · 클러스터 `web-test` · ACR `wleks55555acr`*
