# ci-cd-section-04-ch-01-kubernates-cli

## 쿠버네티스 간단 명령어  
노드 확인  
: `kubectl get nodes`  

파드 확인
: kubectl get pos 

디플로이먼트 확인
: kubectl get deployments
  
서비스 확인
: kubectl get services
  
Nginx 서버 실행
: kubectl run sample-nginx --image=nginx --port=80  
쿠버네티스는 기본적으로 컨테이너가 동작중인 상태에서 동작합니다.  
그리고 기본적으로 `pod`단위로 동작하기 때문에 실행하고자 하는 서비스, 
어떤 미들웨어 , 어떤 운영체제 어플리케이션들이 컨테이너로 패키징되어 있어서 
사용할 수 있는 상태여야합니다.  
Nginx라는 웹서버가 이미지로 제공되고 이걸 `sample-nginx` 파드로 생성하겠다는 의미입니다.
  
컨테이너 정보 확인
: kubectl describe pod/sample-nginx
```Actionscript
root@k8s-master:~# kubectl describe pod/nginx-test
Name:             nginx-test
Namespace:        default
Priority:         0
Service Account:  default
Node:             k8s-node01/192.168.32.11   // 어느 노드에 실행중인지 알수 있습니다.
Start Time:       Fri, 09 Feb 2024 04:34:41 +0000
Labels:           run=nginx-test
Annotations:      cni.projectcalico.org/containerID: cb98c8f28ded8002b6f1bb788f1ee9c74bc04e4ca5bc983c8f200247e7ee4817
                  cni.projectcalico.org/podIP: 10.96.85.193/32
                  cni.projectcalico.org/podIPs: 10.96.85.193/32
Status:           Running  // 동작 상태
IP:               10.96.85.193 //  어느 가상 ip에 할당되었는지 확인할 수 있습니다.
IPs:
  IP:  10.96.85.193
Containers:
  nginx-test:
    Container ID:   containerd://680720bd5a66337cd21583ef3807d3f8fcfb5cc9b3152d1d56d00d8ba852c549
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:84c52dfd55c467e12ef85cad6a252c0990564f03c4850799bf41dd738738691f
    Port:           80/TCP // 포트정보
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 09 Feb 2024 04:34:53 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-29vvd (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-29vvd:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none> // 문제 원인을 확인 할 수 있는 부분
```   

 
파드 삭제
: kubectl delete pod/sample-nginx-XXXXX-XXXXX
  
Scale 변경 (2개로 변경)
: kubectl scale deployment sample-nginx --replicas=2
  
Script 실행
:  kubectl apply -f sample1.yml  
  
## Deployments
가지고 있는 `Pod`를 레플리카 서트라고 해서 여러 개의 형태로 스케일링해서 만들거나 
스케줄링 작업을 할때, 스토리지 작업을 할 때 사용할 수 있는 설치 개념입니다.  
  
```Actionscript
root@k8s-master:~# kubectl create deployment sample-nignx --image=nginx
deployment.apps/sample-nignx created

root@k8s-master:~# kubectl get deployments
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
sample-nignx   1/1     1            1           30s

root@k8s-master:~# kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
sample-nignx-6c6b9bcbb4-pzvcl   1/1     Running   0          48s

root@k8s-master:~# kubectl delete pod/sample-nignx-6c6b9bcbb4-pzvcl
pod "sample-nignx-6c6b9bcbb4-pzvcl" deleted

root@k8s-master:~# kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
sample-nignx-6c6b9bcbb4-nxr77   1/1     Running   0          35s
```  

Pod (파드)
: **Pod는 쿠버네티스에서 가장 기본적인 배포 단위입니다.**
+ Pod는 하나 이상의 컨테이너로 구성되어 있습니다. 주로 여러 개의 컨테이너가 동일한 호스트에 배치되어 특정한 네트워크 및 스토리지를 공유할 때 사용됩니다.
+ 각 Pod는 고유한 IP 주소를 가지며, 포트 공간을 공유합니다. 이는 Pod 내부의 컨테이너 간 통신이나 Pod 외부와의 통신을 위해 사용됩니다.
+ Pod는 일시적이며(stateless) 관리되지 않습니다. 따라서 쿠버네티스는 Pod의 생애주기를 관리하지 않습니다.
  
  
Deployment (디플로이먼트)
: **Deployment는 Pod를 관리하고 유지하는데 사용되는 쿠버네티스 리소스입니다.**  
+ Deployment는 Pod의 상태를 지속적으로 모니터링하고 필요에 따라 새로운 Pod를 생성하거나 기존 Pod를 업데이트합니다.
+ Deployment를 사용하면 롤링 업데이트, 롤백, 스케일링과 같은 작업을 수행할 수 있습니다.
+ Deployment는 Pod의 라이프사이클을 관리하고 안정적으로 운영할 수 있도록 도와줍니다.
+ 간단히 말하자면, Pod는 애플리케이션의 실행 단위이며  
    Deployment는 Pod의 관리와 운영을 담당하는 리소스입니다. Pod는 직접 관리되지 않고 Deployment를 통해 관리되며, Deployment를 통해 애플리케이션을 쉽게 스케일링하고 업데이트할 수 있습니다.  
  
```Actionscript
kubectl scale deployment sample-nginx --replicas=2
```    
이렇게 하면 sample-nginx라는 Deployment가 생성되며 Pod 2개를 가용 노드에서 관리하게 됩니다.  
```Actionscript
kubectl scale deployment sample-nginx --replicas=1
```  
이렇게 작성하면 Pod를 하나만 관리하고 나머지는 Pod는 중지됩니다.  
  
이렇게 작성한 코드를 스크립트를 통해서 관리할 수 있습니다.

## 쿠버네티스 스크립트

**nginx-deployment.yml**
```Actionscript
apiVersion: apps/v1         # 사용하는 쿠버네티스 API 버전을 지정합니다.
kind: Deployment            # 리소스 유형을 지정합니다. 이 경우 Deployment를 생성합니다.
metadata:                   # 리소스의 메타데이터를 정의합니다.
  name: cicd-deployment     # Deployment의 이름을 지정합니다.
  labels:                   # Deployment에 적용될 라벨을 정의합니다.
    app: tomcat              
spec:                       # Deployment의 스펙을 정의합니다.
  replicas: 2               # 생성할 Pod의 개수를 지정합니다. 이 경우 2개의 Pod를 생성합니다.
  selector:                 # Deployment가 관리할 Pod를 선택하는 방법을 지정합니다.
    matchLabels:            # 라벨 기반 선택자를 사용하여 Pod를 선택합니다.
      app: cicd-devops-project            
  template:                 # 새로운 Pod를 생성할 때 사용할 템플릿을 정의합니다.
    metadata:               # Pod 템플릿의 메타데이터를 정의합니다.
      labels:               # Pod에 적용될 라벨을 정의합니다.
        app: cicd-devops-project          
    spec:                   # Pod의 스펙을 정의합니다.
      containers:           # Pod 내에 포함될 컨테이너를 정의합니다.
      - name: cicd-devops-project # 컨테이너의 이름을 지정합니다.
        image: kamser0415/cicd-project-ansible # 사용할 컨테이너 이미지를 지정합니다.
        imagePullPolicy: Always
        ports:              # 컨테이너가 노출할 포트를 지정합니다.
        - containerPort: 8080 # 컨테이너가 80번 포트를 노출합니다.
```  
yml 파일 실행하기
```Actionscript
kubectl apply -f nginx-deployment.yml
```

파트 상세 정보
```Actionscript
$ kubectl get pods -o=wide
NAME                                READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
nginx-deployment-86dcfdf4c6-hcwq9   1/1     Running   0          18m   10.96.85.196   k8s-node01   <none>           <none>
nginx-deployment-86dcfdf4c6-hzcdr   1/1     Running   0          18m   10.96.58.194   k8s-node02   <none>           <none>
```  
기본적으로 파드는 공평하게 각 노드에 분배됩니다.  
특정한 노드에 지정을 하거나, 특정한 노드에 분배가 안되도록 설정도 가능합니다.  
  
해당 정보를 통해서 Pod도 마찬가지로 `exec -it` 커맨드로 접속할 수 있습니다.  
```Actionscript
root@k8s-master:~# kubectl exec -it nginx-deployment-86dcfdf4c6-hzcdr -- /bin/bash
root@nginx-deployment-86dcfdf4c6-hzcdr:/# 
```    

**Deployment 포트-포워딩하기**  
```Actionscript
kubectl expose deployment nginx-deployment --port=80 --type=NodePort
```  
service에 등록하는 방법은 cli로 작성하거나 스크립트 파일로도 작성할 수 있습니다.
**Service.yml**
```Actionscript
apiVersion: v1
kind: Service
metadata:
  name: cicd-service    # 서비스의 이름
  labels:
    app: cicd-devops-project   # 서비스에 할당된 라벨
spec:
  selector:
    app: cicd-devops-project   # 서비스가 연결할 파드를 선택하기 위한 셀렉터
  type: NodePort   # 서비스 유형 (NodePort)
  ports:
    - port: 8080   # 서비스가 노출할 포트
      targetPort: 8080   # 서비스가 연결할 파드의 포트
      nodePort: 32000   # 노드에 노출될 포트 (NodePort 유형에서만 사용됨)
```  

```sql
 -- 대략적인 그림
                               +--------------+
                               |  External    |
                               |  Client      |
                               +------+-------+
                                      |
                                      v
                               +--------------+
          External              |   NodePort   |
          Network               |   Service    |
                               +------+-------+
                                      |
                                      v
                               +--------------+
                               |   Cluster    |
                               |   IP:Port    |
                               |   Service    |
                               |              |
                               | targetPort   |
                               +------+-------+
                                      |
                                      v
                               +--------------+
                               |    Pod       |
                               |              |
                               |  Container   |
                               |              |
                               |  targetPort  |
                               +--------------+
```

nodePort
: 클러스터의 각 노드에서 서비스에 접근하기 위한 포트입니다. 클러스터 외부에서 해당 포트로 접근하면 서비스로 트래픽이 전달됩니다.

port
: 서비스가 노출하는 포트입니다. 이 포트로 클러스터 내부에서 서비스에 접근할 수 있습니다.  
  
targetPort
: 서비스가 연결하는 파드의 포트입니다. 서비스로 들어온 트래픽은 해당 파드의 이 포트로 전달됩니다.  
  
```Actionscript
root@k8s-master:~# kubectl get service
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
cicd-service   NodePort    10.109.200.43   <none>        8080:32000/TCP   33m
```  

