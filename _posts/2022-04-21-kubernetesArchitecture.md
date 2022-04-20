---
layout: post
author: Bruce Lee
title: Kubernetes 아키텍쳐 및 동작원리 1
---

👨‍🎓 쿠버네티스 아키텍쳐

쿠버네티스는 클러스터에는 Control Plane과 workker node 가 있다.<br/><br/>
(1) 컨트롤 플레인 (마스터 노드)
마스터 노드는 클러스터를 통제하고 관리하는 쿠버네티스의 중심부이며 아래와 같은 구성요소가 있다.<br/><br/>

> API server<br/>
client와 통신하는 쿠버네티스 API 서버로 쿠버네티스 구성요소들간의 통신은 대부분 API서버를 거친다.<br/>
Scheduler<br/>
api서버가 노드들에게 일을 할당할때 직접시키는게 아니라 스케쥴러에게 할당한다.<br/>
스케줄러는 애플리케이션을 노드에 할당하는 기능을 맡는다.<br/>
etcd<br/>
분산 저장소이며 안드로이드에서 자주 사용되는 sqlite를 사용하기도 한다.<br/>
Controller Manager<br/>
쿠버네티스는 선언형 api를 사용한다.<br/>
Controller Manager의 요소들은 선언된 API에 맞게 쿠버네티스 resource들을 생성, 복제, 관리 한다.(쿠버네티스 비즈니스 로직이 담긴 부분)

<br/><br/>
(2) 워커노드<br/><br/>
- kubelet<br/>
: 스케줄러에서 워커노드들에게 애플리케이션을 할당할때 kubelet에 요청한다. kubelet은 pod에서 컨테이너가 동작하도록 하고, 팟을 관리한다.<br/>
- kube-proxy<br/>
: 네트워크 관련 규칙을 유지해주는 곳이다.<br/><br/>


💡 **쿠버네티스 동작원리**<br/>
- 쿠버네티스는 선언형 api를 사용하는 특징이 있다.<br/>
- 쿠버네티스는 선언형 api를 사용하여 내용이 etcd에 저장되고 etcd를 모니터링하던 controller에 의해 동작한다.<br/>

🎡 쿠버네티스 동작순서
(1) 클라이언트는 쿠버네티스 api server에 요청<br/>
: curl이나 postman 등을 사용하여 http request를 날릴수도 있지만 일반적으로 kubectl을 사용하여 요청한다.<br/>
(2) etcd에 api의 내용 저장<br/>
: api server에 선언형 api로 요청이 들어오면 선언형 api의 특징답게 바로 컨테이너를 생성하라는 명령을 동작시키지 않는다. 분산 저장소인 etcd에 들어온 내용을 저장한다.<br/>
(3) etcd를 감시하던 controller 동작<br/>
: controller는 etcd에 내가 담당하고있는 resource가 들어왔는지를 감시하다가 etcd에 자신의 역할에 맞는 내용이 저장되어있다면 스케줄러에게 동작을 요청한다. 쿠버네티스의 비즈니스 로직은 controller에 숨어있다.<br/>
(4) 스케줄러 동작<br/>
: 스케줄러는 워커노드의 kubelet과 통신한다.<br/><
(5) kubelet 동작<br/>
: kubelet은 노드에 pod 등을 생성합니다.<br/>

1. Pod이란<br/>
: Pod 는 쿠버네티스에서 관리하는 가장 작은 배포 단위이다.<br/>
쿠버네티스는 여러 컴퓨터에 흩뿌려져있는 컨테이너를 관리하도록 도와주는 플랫폼이지만, 쿠버네티스의 가장 작은 배포단위는 컨테이너가 아닌 pod이다.<br/>
pod안에 컨테이너 (여러개가 존재할 수도 있음) 존재하는 구조이다.<br/>
쿠버네티스의 마스터 노드에서 명령이 떨어지면 워커노드에 팟(컨테이너포함)을 생성하는게 기본 동작이다.<br/><br/>

2. pod 생성하기<br/>
- 먼저 8080포트 /에 http request를 보내면 hello world 문자열을 내려주는 백엔드 애플리케이션을 실행하는 이미지를 만들어준다.<br/>
spring으로 만들어진 이미지를 사용해보자.<br/><br/>

## pod는 보통 두가지 방법으로 만든다.<br/><br/>
(1) kubectl run<br/>
: kubectl run 명령어를 통해 pod를 생성할 수 있습니다.<br/>
### run 명령어를 통해 pod를 생성하고 image를 사용한다.<br/>
$ kubectl run mypod --image=(image name:version)<br/>

(2). yaml파일로 상세스펙을 정의하여 생성<br/>
>apiVersion: v1<br/>
kind: Pod<br/>
metadata:<br/>
name: mypod<br/>
spec:<br/>
containers:<br/>
> - name: mypod<br/>
>  image: (image name):version<br/>
>  ports:<br/>
>  - containerPort: 8080<br/>
>    protocol: TCP<br/>
>    쿠버네티스의 yaml파일은 크게 apiVersion, kind, metadata, spec으로 구성된다.<br/>
>    다음과 같이 yaml파일을 생성하고 kubectl apply 명령어를 사용하여 쿠버네티스에 적용한다.<br/>

$ kubectl apply -f mypod.yaml<br/>
: 이제 쿠버네티스로 연결된 여러 컴퓨터중 한군데에는 팟(컨테이너)가 올라가게 된다.<br/>
위처럼 yaml파일을 작성해도 되지만 아래와같은 명령어를 작성하여 생성하기전 yaml파일을 뽑아낸후 작성할 수도 있으며<br/>
$ kubectl run mypod --image=(repo name)/(image name):version --dry-run=client > mypod.yaml<br/>
처음 만드는것이 아니며 수정이 아래와같은 명령어로 yaml파일을 뽑아내 수정후 다시 apply 하거나 edit 명령어로 수정이 가능하다.
$ kubectl get pod mypod -o yaml > mypod.yaml<br/>
$ kubectl edit pod mypod<br/>

3. pod 확인하기<br/>
(1). pod 생성 확인하기<br/>
### pod의 생성 및 상태를 확인한다.<br/>
$ kubectl get pod<br/>
NAME READY STATUS RESTARTS AGE<br/>
pod/mypod 1/1 Running 0 57m<br/>
쿠버네티스 cli 클라이언트인 kubectl을 가장 대표하는 명령어는 kubectl get {resource} 이다.<br/>
해당 명령어를 통해 pod 같은 리소스를 확인할 수 있다.<br/>
pod이 정상적으로 올라왔는지 확인할때는 $ kubectl get pod를 입력해준다.<br/>
(2) describe 명령어를 사용해 pod의 상세내역을 확인하기<br/>
describe 명령어를 사용해 pod의 상세내역을 확인할 수 있다.<br/>

### pod의 상태를 자세히 본다.<br/>
$ kubectl describe pod mypod<br/>
### 보통 아래쪽의 이벤트를 확인합니다.<br/>
$ kubectl describe pod mypod<br/>
> Name:         mypod<br/>
Namespace:    default<br/>
Priority:     0<br/>
Node:         minikube/192.168.49.2<br/>
Start Time:   Sat, 19 Jun 2021 23:18:27 +0900<br/>
Labels:       run=mypod<br/>
Annotations:  <none><br/>
Status:       Running<br/>
IP:           172.17.0.3<br/>
IPs:<br/>
IP:  172.17.0.3<br/>
Containers:<br/>
mypod:<br/>
Container ID:   docker://7db35f18b7d4f298e82c215a83745e22ebbbc784662424b04496a6da03b0ec6f<br/>
Image:          jaeho310/helloworld:1<br/>
Image ID:       docker-pullable://jaeho310/helloworld@sha256:c227f6acd0b6ec6e0329f484eeb4725bf8263fa6e6d0edb0d4412a026e414dfc<br/>
Port:           <none><br/>
Host Port:      <none><br/>
State:          Running<br/>
Started:      Sat, 19 Jun 2021 23:18:28 +0900<br/>
Ready:          True<br/>
Restart Count:  0<br/>
Environment:    <none><br/>
Mounts:<br/>
/var/run/secrets/kubernetes.io/serviceaccount from default-token-g89bx (ro)<br/>
Conditions:<br/>
Type              Status<br/>
Initialized       True<br/>
Ready             True<br/>
ContainersReady   True<br/>
PodScheduled      True<br/>
Volumes:<br/>
default-token-g89bx:<br/>
Type:        Secret (a volume populated by a Secret)<br/>
SecretName:  default-token-g89bx<br/>
Optional:    false<br/>
QoS Class:       BestEffort<br/>
Node-Selectors:  <none><br/>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s<br/>
node.kubernetes.io/unreachable:NoExecute op=Exists for 300s<br/>
Events:<br/>
Type    Reason     Age   From               Message<br/>
>  ----    ------     ----  ----               -------<br/>
>Normal  Scheduled  17s   default-scheduler  Successfully assigned default/mypod to minikube<br/>
>Normal  Pulled     16s   kubelet            Container image "jaeho310/helloworld:1" already present on machine<br/>
>Normal  Created    16s   kubelet            Created container mypod<br/>
>Normal  Started    16s   kubelet            Started container mypod<br/>


(3) 쿠버네티스 클러스터의 이벤트로그로 확인하기<br/>
pod을 여러개 생성한경우 get event 명령어로 전체 pod를 확인할 수 있다.<br/>
> $ kubectl get event<br/>
$ kubectl get event<br/>
LAST SEEN   TYPE     REASON                    OBJECT          MESSAGE<br/>
106s        Normal   Starting                  node/minikube   Starting kubelet.<br/>
106s        Normal   NodeHasSufficientMemory   node/minikube   Node minikube status is now: NodeHasSufficientMemory<br/>
106s        Normal   NodeHasNoDiskPressure     node/minikube   Node minikube status is now: NodeHasNoDiskPressure<br/>
106s        Normal   NodeHasSufficientPID      node/minikube   Node minikube status is now: NodeHasSufficientPID<br/>
106s        Normal   NodeAllocatableEnforced   node/minikube   Updated Node Allocatable limit across pods<br/>
93s         Normal   Starting                  node/minikube   Starting kube-proxy.<br/>
82s         Normal   RegisteredNode            node/minikube   Node minikube event: Registered Node minikube in Controller<br/>
5d22h       Normal   Killing                   pod/mypod       Stopping container mypod<br/>
48s         Normal   Scheduled                 pod/mypod       Successfully assigned default/mypod to minikube<br/>
47s         Normal   Pulled                    pod/mypod       Container image "jaeho310/helloworld:1" already present on machine<br/>
47s         Normal   Created                   pod/mypod       Created container mypod<br/>
47s         Normal   Started                   pod/mypod       Started container mypod<br/>

<br/><br/>
(4) pod 컨테이너의 애플리케이션 로그 확인하기<br/>
: kubectl logs 명령어를 통해 확인한다.<br/>
해당 로그는 스프링 애플리케이션이 시작될때 뜨는 로그이다.<br/>
$ kubectl logs mypod<br/>

.   ____          _            __ _ _<br/>
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \<br/>
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \<br/>
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )<br/>
'  |____| .__|_| |_|_| |_\__, | / / / /<br/>
=========|_|==============|___/=/_/_/_/<br/>
:: Spring Boot ::                (v2.5.1)<br/>
<br/>
2021-06-19 14:18:30.970  INFO 1 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication v0.0.1-SNAPSHOT using Java 11.0.1 on mypod with PID 1 (/app.jar started by root in /)<br/>
2021-06-19 14:18:30.973  INFO 1 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default<br/>
2021-06-19 14:18:32.911  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)<br/>
2021-06-19 14:18:32.932  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]<br/>
2021-06-19 14:18:32.933  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.46]<br/>
2021-06-19 14:18:33.031  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext<br/>
2021-06-19 14:18:33.031  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1959 ms<br/>
2021-06-19 14:18:34.035  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''<br/>
2021-06-19 14:18:34.050  INFO 1 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 4.02 seconds (JVM running for 5.485)<br/>
<br/>

(5) pod 컨테이너에 직접 접근해 curl 명령어로 확인하기<br/>
# pod의 컨테이너에 접근한다.<br/>
$ kubectl exec mypod -it -- sh<br/>
# pod에 접근했다면 curl 명령어로 웹 애플리케이션이 동작하는지 확인한다.<br/>
$ curl get -x 127.0.0.1:8080<br/>
# response로 hello world 를 내려주는것을 확인한다.<br/>
8080포트 "/" 에 hello world를 내려주도록 했으므로 확인한다.<br/>
<br/>
쿠버네티스 service라는 리소스를 공부하기 이전이므로 브라우저에서는 접근할 수 없다.<br/>
pod안에 접속해 http request를 보내 확인한다.<br/>

4. pod를 삭제하기<br/>
## 이름으로 삭제한다.<br/>
$ kubectl delete pod mypod<br/>
### 연습용 pod를 많이 생성했다면 모두 삭제한다.<br/>
### 네임스페이스를 지정한적이 없으므로 모든 resource는 default 네임스페이스에 올라간다.<br/>
### default 네임스페이스에 있는 모든 pod을 삭제할 수도 있다.<br/>
$ kubectl delete pod --all -n default<br/>
$ kubectl delete {resource} 명령어를 사용해서 쿠버네티스 리소스를 삭제할 수 있다.<br/>
삭제할때는-A --all 등을 사용해서 한번에 삭제할 수 있으나,<br/>
all이 어떤 부분을 의미하는지 정확하게 확인한 상태가 아니면 운영쪽에서는 사용을 안하는게 좋다.<br/>