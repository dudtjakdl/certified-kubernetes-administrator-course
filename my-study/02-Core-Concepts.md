# Section 2: Core Concepts

## 11. Cluster Architecture

쿠버네티스의 목적 : 응용프로그램을 자동화된 컨테이너 형식으로 호스트

- ETCD cluster : 키 값 형식으로 정보를 저장하는 데이터베이스

- kube-scheduler : 컨테이너를 적재할 노드를 식별, 컨테이너 리소스 requirements나 노드 capacity, 다른 정책이나 제약조건들, taints와 toleration에 근거함
- Contriller-Manager

- Kube API : 클러스터 내에서 모든 작업을 오케스트레이션, 주기적으로 Kubelet으로부터 상태보고서를 가져옴(모니터 역할로)

- 컨테이너 런타입 엔진 : Docker - 클러스터 내 모든 노드에 설치(마스터 노드 포함)
- kubelet : 클러스터의 각 노드에서 실행되는 agent, Kube API 서버의 지시를 듣고 노드에서 컨테이너를 배포하거나 파괴함 (배의 선장 역할)

- kube-proxy : 작업자 노드 간(클러스터 내부의 서비스 간)의 통신, 작업자 노드에 필요한 규칙 시행되도록

## 12. Docker-vs-ContainerD

- 컨테이너 런타임 인터페이스 (CRI)
- Open Container Initiative (OCI)
  - imagespec : 이미지 빌드 방식에 대한 기준 정의
  - runtimespec

- dockershim : 컨테이너 런타임 인터페이스 밖에서 Docker를 지속 지원하는 임시방편
- containerD : 도커의 일부지만 현재는 독립된 프로젝트, 도커 설치할 필요없이 컨테이너 자체 설치 가능

- ctrl : 디버깅 컨테이너만을 위해 만들어진 명령어, 사용자 친화적 X
- nerdctl : 도커가 지원하는 대부분의 옵션 지원, 컨테이너드 커뮤니티에서 왔고 컨테이너드를 위한 CLI
- crictl : CRI 호환 가능한 컨테이너 런타임과 상호작용 하는데 사용, 디버깅 목적으로만 사용됨 (kubelet과 잘 어울림),  쿠버네티스 커뮤니티에서 옴

## 13. ETCD For Beginners

```shell
> ./etcdctl set key1 value1
```

```shell
> ./etcdctl get key1
```

ETCDCTL은 API버전 2와 3이 있는데 디폴트는 2버전

## 14. ETCD in Kubernetes

## 16. Kube-API Server

1. Authenticate User
2. Valudate Request
3. Retrive data
4. Update ETCD
5. Scheduler
6. Kubelet

 데이터 저장소와 직접 상호작용하는 유일한 구성요소

## 17. Kube Controller Manager

1. Watch Status
2. Remediate Situation

- Node-Controller : kube-apiserver를 통해 node 상태 모니터링

  - Node Monitor Period = 5s
  - Node Monitor Grace Period = 40s
  - POD Eviction Timeout = 5m

- Replication-Controller : 복제품 세트의 상태를 모니터링하고 원하는 수의 팟이 세트 내에서 항상 사용 가능하도록 유지

- 그 외 다양한 컨트롤러 존재

- **Kube-Controller-Manager** : 마스너 노드 위에 kube-system namespace에 POD로써 존재

  ```shell
  > kubectl get pods -n kube-system
  ```

## 18. Kube Scheduler

어떤 pod를 어떤 node에 넣을지 결정 (결정만 함!! pod를 node에 넣는 작업은 하지 않음->이건 kubelet 역할)

1. Filter Nodes : 스케줄러가 해당 pod에 맞지 않는 node를 걸러냄 (예시: 노드의 CPU나 메모리 리소스가 부족할때)

2. Rank Nodes : 걸러진 node를 뺀 나머지 node를 우선순위 함수를 이용해 0에서 10까지 점수를 매김

- 기준이 될 수 있는 것들
  - Resource Requirements and Limits
  - Taints and Tolerations
  - Node Selectors/Affinity

## 19. Kubelet

1. Register Node
2. Create PODs
3. Monitor Node & PODs

worker 노드에 반드시 수동으로 kubelet 설치해야함!!

## 20. Kube Proxy

- POD Network : 내부 가상 네트워크. 모든 pod가 연결되는 클러스터 내 모든 node에 걸쳐 있음

- Kube-proxy : 클러스터의 각 노드에서 실행되는 프로세스, 새 서비스가 생성될 때마다 각 노드에 적절한 규칙을 만들어 그 서비스(=백엔드 pod)로 트래픽 전달
  iptables 규칙을 만들어 서비스의 ip로 향하는 트래픽을 향하게 함
- 각 노드에 kube-proxy 파드로 배포 (daemonset으로 배포)

## 21. Recap - Pods

- 컨테이너는 pod라고 불리는 쿠버네티스 오브젝트로 캡슐화됨
- pod는 애플리케이션의 단일 인스턴스
- pod는 쿠버네티스에서 생성할 수 있는 가장 작은 오브젝트임

클러스터 > 노드 > POD(인스턴스) > Docker 컨테이너(POD로 캡슐화됨) > 응용 프로그램

-> 스케일업 할때 노드 안에 POD를 늘림

-> 노드의 capacity가 꽉 찼을 때는??

-> 클러스터 안에 노드를 늘림

- pod는 보통 컨테이너와 1대1 관계 
  - 물론 예외는 있음, 그러나 pod 안에 같은 응용프로그램 컨테이너가 여러개이기보다는 서로 다른 응용프로그램 컨테이너인 경우가 많음 (로컬 호스트)

```shell
> kubectl run nginx --image nginx 
```

```shell
> kubectl get pods
```

## 22. Pods with YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
	labels: # 딕셔너리로 원하는만큼 키-값 쌍 추가 가능
		app: myapp
		type: front-end
spec: # 생성하려는 개체들에 따라 입력해야할 속성이 다름(pod는 containers)
	containers: # List/Array(여러 개의 컨테이너가 pod 안에 존재할 수 있기 때문)
		- name: nginx-container # 대시(-)는 리스트의 첫번째 항목을 가리킴
		  image: nginx # 저장소에 있는 도커 이미지
```

| Kind       | Version |
| ---------- | ------- |
| POD        | v1      |
| Service    | v1      |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |

```
-- pod 생성
> kubectl create -f [파일이름].yml

-- 사용 가능한 pod 목록 확인
> kubectl get pods

-- pod 상세정보 조회
> kubectl describe pod [POD NAME]

-- pod 삭제
> kubectl delete pod [POD NAME]
```

## 23.Demo - Pods with YAML

## 27. Practice Test - Pods

 https://uklabs.kodekloud.com/topic/practice-test-pods-2/

```
> kubectl run nginx --image=nginx

> kubectl get pods -o wide

-- dry-run 옵션으로 실제 명령을 실행하지 않고 결과 값만 확인, -o yaml 을 통해서 yaml 형태로 결과 출력
> kubectl run redis --image=redis123 --dry-run -o yaml

> kubectl run redis --image=redis123 --dry-run=clinet -o yaml

-- 이미 create 된 것 다시 적용할 때
> kubectl apply -f redis.yaml
```

![image-20240522232449025](C:\Users\yskim\AppData\Roaming\Typora\typora-user-images\image-20240522232449025.png)

## 29. Recap - ReplicaSets

- Replication Controller 

  - 쿠버네티스 클러스터에 있는 단일 pod의 다중 인스턴스를 실행하도록 고가용성 제공

    pod가 하나여도 사용 가능, 기존의 pod가 다운 됐을 때 자동으로 새 pod를 불러옴

    특정 pod가 항상 실행되도록 보장

  - 클러스터 내 여러 노드에 걸쳐 있음
  - 수요가 증가했을 때 앱 스케일 조정

  ```yaml
  apiVersion: v1
  kind: ReplicationController
  metadata: # Replication Controller 
  	name: myapp-rc
  	labels:
  		app: myapp
  		type: front-end	
  spec: # Replication Controller 
  	template: # POD 정의 yaml 파일 복사
  		metadata: # POD
              name: myapp-pod
              labels:
                  app: myapp
                  type: front-end
          spec: # POD
              containers:
                  - name: nginx-container
                    image: nginx
                    
  	replicas: 3
  ```

  

- Replica Set

  - Replication Controller와 비슷하지만 같진 않음, 좀 더 새로운 권장 방법
  - pod를 모니터링 하면서 하나가 다운되면 새 pod 배포
  - labels로 어떤 pod을 모니터링할지 판단함 (Replication Controller 와의 차이점)

```yaml
apiVersion: apps/v1
kind: ReplicationSet
metadata:
	name: myapp-replicaset
	labels:
		app: myapp
		type: front-end
spec:
	template:
		metadata:
            name: myapp-pod
            labels:
                app: myapp
                type: front-end
        spec:
            containers:
                - name: nginx-container
                  image: nginx
   
	replicas: 3
	selector:
		matchLabels:
			type: front-end
```

replicas (복제 개수) 를 변경하는 방법

```
-- yaml 파일 수정 후 대체 명령
> kubectl replace -f [파일이름].yaml

> kubectl scale --replicas=6 -f [파일이름].yaml
> kubectl scale --replicas=6 replicaset [replicaset NAME]
```

| DESIRED                | CURRENT                  | READY                           |
| ---------------------- | ------------------------ | ------------------------------- |
| replicas에 설정한 개수 | 실제 동작중인 pod의 개수 | 사용할 준비가 완료된 pod의 개수 |



```
-- 이미 정의된 오브젝트 수정
> kubectl edit [TYPE] [NAME]

-- 타입 설명 조회
> kubectl explain [TYPE]
```



