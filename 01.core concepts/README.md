## 기본 개념

![image](https://github.com/saechimdaeki/Dev-Diary/assets/40031858/1f0a9ed2-3b4a-47b4-90fa-70f0e98d62df)


- `마스터노드`는 쿠버네티스 클러스터를 관리하고 서로 다른 노드에 대한 정보를 저장하고 어떤 컨테이너가 어디로 갈지 계획하고

    노드와 컨테이너를 모니터링하는 등의 책임을 지님. 
- 많은 컨테이너에 대한 정보가 언제 적재되었는지 등을 Etcd라는 고가용 키 값 스토어에 저장된다.
- `스케줄러`는 컨테이너를 설치하기 위해 올바른 노드를 식별한다. 컨테이너 리소스 요구 사항이나 워커 노드 용량 혹은 정책이나 제약조건들에 근거해서
- `노드 컨트롤러`가 노드를 관리하고. 새 노드를 클러스터에 온보딩하고 노드가 사용불가능하거나 파괴되는 상황을 처리한다.
- `복제 컨트롤러`는 원하는 컨테이너 수가 복제 그룹에서 항상 실행되도록 보장함.
- `kube-apiServer`는 쿠버네티스의 주요 관리 구성요소로 클러스터 내에서 모든 작업을 오케스트레이션 함.
- 클러스터에서 관리 작업을 수행하는 데 외부 사용자가 사용하는 것.

컨테이너는 어디에나 있으니 모든게 컨테이너와 맞아야 한다. 어플리케이션은 컨테이너 형태이고 마스터 노드에의해 전체 관리된다.

`DNS 서비스 네트워킹 솔루션`은 컨테이너 형태로 배포될 수 있다. 그래서 이런 컨테이너를 실행할 소프트웨어가 필요하다

이게 컨테이너 런타임엔진인데 `Docker`가 대표적이다. 

`kubelet`은 클러스터의 각 노드에서 실행되는 에이전트이다. kube api서버의 지시를 듣고 필요한대로 노드에서 컨테이너를 배포하거나 파괴한다

`kubeAPI서버`는 주기적으로 kubelet으로부터 정보를 가져와 컨테이너의 상태를 모니터링한다

`kube-proxy서비스`는 작업자 노드에 필요한 규칙이 시행되도록한다. 그 위에서 시행되는 컨테이너가 서로 닿을 수 있도록.


#### 요약
- 마스터 노드에 ETCD Cluster가 있어서 클러스터에 관한 정보를 저장한다
- 큐브 스케줄러가 있는데 노드의 응용프로그램이나 컨테이너의 스케줄을 짜는 역할을 한다 
- kube controller manager는 컨트롤러도 다르고 기능도 다른 노드 컨트롤러, 복제 컨트롤러 등을 다룬다.
- kube Apiserver는 클러스터 내에서 모든 작업을 오케스트레이션 한다
- 워커노트에는 kubelet이 있는데 kube api서버의 지시를 듣고 컨테이너와 kube프록시를 관리한다 

    클러스터 내부의 통신을 가능하게 한다.

---

## `ETCD`

ETCD: 분산되고 신뢰할 수 있는 스토어. 단순하고 안전하며 신속하다. (key-value store)

<img width="976" alt="image" src="https://github.com/saechimdaeki/Dev-Diary/assets/40031858/cf4df8a0-81c1-40c6-b422-e69c8d1451dc">

##### Install ETCD

1. Downlad

```bash
curl -L https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz -o etcd-v3.3.11-linux-amd64.tar.gz
```

2. Extract 

```bash
tar xzvf etcd-v3.3.11-linux-amd64.tar.gz
```

3. Run ETCD

```bash
./etcd-v3.3.11-linux-amd64/etcd
```

```bash
./etcdctl put key1 value1

./etcdctl get key1
-> value1
```

#### ETCD in Kubernetes

이 데이터 저장소는 클러스터에 관한 정보를 저장한다.
- Nodes
- PODs
- Configs
- Secrets
- Accounts
- Roles
- Bindings
- Others

## `Kube-API Server`

```bash
kubectl get nodes
```

kubectl를 실행하면 kube-apiserver에 도달한다. kube-apiserver가 먼저 요청을 인증하고 유효성을 확인한다

그러면 ETCD Cluster에서 데이터를 회수해 요청된 정보로 응답한다

pod을 생성하는 예시를 보자
- 요청이 먼저 인증되고 그 다음 유효성이 확인된다
- 이 경우 kube-apiserver는 노드에 할당하지 않고 pod객체를 생성한다 
- ETCD Cluster에 있는 정보를 업데이트하고 pod가 생성된 사용자를 업데이트한다
- 스케줄러는 지속적으로 kube-apiserver를 모니터링하고 노드가 할당되지 않은 새로운 pod가 있다는 것을 알게된다
- 스케줄러가 올바른 노드를 식별해 새 pod을 키고 다시 kube-apiserver와 통신한다
- kube-apiserver는 해당하는 클러스터 정보를 업데이트한다
- kube-apiserver는 해당 정보를 적절한 worker nodes에서 kubelet에 전달한다
- kube-apiserver는 노드에 pod을 생성하고 컨테이너 런타임 엔진에 지시해 앱 이미지를 배포한다 
- 완료되면 kublet은 상태를 kube-apiserver로 다시 업데이트하고 kube-apiserver는 ETCD cluster에서 데이터를 업데이트한다
- 변화를 요구할때마다 비슷한 패턴이 반복된다

kube-apiserver는 클러스터 변경을 위해 수행해야하는 모든 작업의 중심에 있다

요약하자면
1. Authenticate User
2. Validate Request
3. Retrieve data
4. Update ETCD
5. Scheduler
6. Kubelet
