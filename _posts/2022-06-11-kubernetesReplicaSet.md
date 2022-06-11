---
layout: post
author: Bruce Lee
title: Kubernetes 레플리카셋과 디플로이먼트
---

👨‍🎓 레플리카셋이란?
<br/>쿠버네티스에서 pod가 죽었을때 다시 복구할 수 있는 이유는 replica로 시작하는 녀석들 덕분이다.
<br/>pod을 복구해주는 resource는 대표적으로 레플리케이션 컨트롤러(replication-controller)와 레플리카셋(replicaset)이 있다.
<br/>리소스를 감시하는 컨트롤러가 pod에 변화가 생겼는지를 감지하고, pod이 죽었을때 다시 복구한다.
<br/>deployment때문에 단독으로 replicaset을 사용하는일은 거의 없지만 매우 중요한 resource이다.
<br/><br/>

2. 레플리케이션 컨트롤러와 레플리카셋
<br/>레플리케이션 컨트롤러는 레플리카셋의 하위호환이다.
<br/>낮은 쿠버네티스 버전에서 레플리케이션 컨트롤러를 사용했다.
<br/>레플리케이션 컨트롤러는 관리할 pod를 selector를 직접 사용해서 선택하고, 레플리카셋은 matchlabels를 사용하여 관리한다.
<br/>matchlabel이 생기면서 여러개의 pod를 취급할 수 있으며 라벨의 key만 갖고있어도, 일치하지 않는 라벨까지도 관리할 수 있다.
<br/><br/>
3. replicaset 구성
(1) 내가 관리할 pod을 선택하기위한 matchlabels<br/>
(2) 내가 관리할 pod의 복제본 수<br/>
(3) 내가 관리할 pod의 명세<br/>
yaml파일에서 이 세가지를 확인할 수 있다.
<br/>
4. replicaset.yaml<br/>
<pre>   apiVersion: apps/v1<br/>
   kind: ReplicaSet<br/>
   metadata:<br/>
    name: myrs<br/>
   spec:<br/>
       replicas: 3<br/>
       selector:<br/>
           matchLabels:<br/>
               app: myHelloWorld<br/>
       template:<br/>
           metadata:<br/>
               labels:<br/>
                   app: myHelloWorld<br/>
       spec:<br/>
           containers:<br/>
            - name: myapp<br/>
              image: repo/helloworld:1<br/>
</pre>
<br/><br/>
<br/>
kubernetes의 yaml파일은 크게 apiVersion, kind, metadata, spec으로 구성되어있다.<br/>
내가 관리할 pod의 명세부터 찾아보면 template 에 있다.
<br/>
spec.templete은 apiversion과 kind만 없어진 pod이다.
<br/>
단 metadata의 labels의 key, value는 정확히 하고 가야합니다.(app: myHelloworld는 그냥 key와 value이다)
<br/>
(2)번 내가 관리할 pod의 복제본 수는<br/>
replicas: 3 이다.<br/>
복제본(pod)을 3개 만든다는 의미이다.<br/>
내가 관리할 pod을 선택하기위한 matchlabels 은<br/>
matchlabels이다. pod에서 만들어줬던 label과 동일하게 만들어줘서 이 pod을 관리할꺼라고 명시한다.<br/>
kind는 replicaset을 쓰니까 replicaset이고 metadata에는 이름, 라벨, 네임스페이스 등이 들어가는데 현재는 name만 명시하였다.<br/>
      <br/><br/>
5. 확인하기<br/>
   (1) apply 명령어를 사용하여 yaml을 적용한다.<br/>
   $ kubectl apply -f replicaset.yaml<br/>
   replicaset.apps/myrs created<br/>
   (2) 조회해서 pod 확인하기<br/>
   $ kubectl get all<br/>
   NAME             READY   STATUS    RESTARTS   AGE<br/>
   pod/myrs-rftgd   1/1     Running   0          21s<br/>
   pod/myrs-sdcws   1/1     Running   0          21s<br/>
   pod/myrs-ww7cb   1/1     Running   0          21s<br/>
   <br/>
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE<br/>
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   8d<br/>
   <br/>
NAME                   DESIRED   CURRENT   READY   AGE<br/>
replicaset.apps/myrs   3         3         3       22s<br/>
replicaset 한개와 pod가 세개 생겼다.
   <br/>
pod의 이름은 replicaset의 이름뒤에 -hash로 이어 붙는다.<br/>
   <br/><br/>
(3) pod을 지우고 조회하기<br/>
$ kubectl delete pod myrs-rftgd<br/>
pod "myrs-rftgd" deleted<br/>
   <br/>
$ kubectl get all<br/>
NAME             READY   STATUS    RESTARTS   AGE<br/>
pod/myrs-cwjbb   1/1     Running   0          20s<br/>
pod/myrs-sdcws   1/1     Running   0          2m57s<br/>
pod/myrs-ww7cb   1/1     Running   0          2m57s<br/>
   <br/>
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE<br/>
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   8d<br/>
   <br/>
NAME                   DESIRED   CURRENT   READY   AGE<br/>
replicaset.apps/myrs   3         3         3       2m57s<br/>
지워졌다는 로그는 나왔지만 hash값이 변한 하나의 pod이 새로 생겨서 pod의 숫자는 그대로 3개이다.<br/>



(4) replicaset의 동작 확인하기<br/>
$ kubectl get event<br/>
# or<br/>
$ kubectl describe replicaset myrs<br/>
<br/>
둘중 하나의 명령어로 확인한다.<br/>
<br/><br/>
> Events:<br/>
Type    Reason            Age    From                   Message<br/>
  ----    ------            ----   ----                   -------<br/>
Normal  SuccessfulCreate  4m58s  replicaset-controller  Created pod: myrs-rftgd<br/>
Normal  SuccessfulCreate  4m58s  replicaset-controller  Created pod: myrs-ww7cb<br/>
Normal  SuccessfulCreate  4m58s  replicaset-controller  Created pod: myrs-sdcws<br/>
Normal  SuccessfulCreate  2m21s  replicaset-controller  Created pod: myrs-cwjbb<br/>
<br/><br/>

pod을 지우자마자 replicaset-controller(replicaset을 관찰하는 녀석)가 pod을 다시 만들어준걸 확인할 수 있다.<br/>

(5) replicaset 지우기<br/>
# replicaset은 rs로 줄여서 쓴다.<br/>
$ kubectl delete rs myrs<br/>
replicaset.apps "myrs" deleted<br/>
<br/><br/>
$ kubectl get all<br/>
NAME             READY   STATUS        RESTARTS   AGE<br/>
pod/myrs-cwjbb   0/1     Terminating   0          18m<br/>
pod/myrs-sdcws   1/1     Terminating   0          20m<br/>
pod/myrs-ww7cb   0/1     Terminating   0          20m<br/>
<br/>
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE<br/>
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   8d<br/>
replicaset을 지우면 pod도 같이 사라지는걸 확인할 수 있다.<br/>

👨‍🎓 deployment
###1. **디플로이먼트**
<br/>deployment가 등장하기 이전 레플리케이션 컨트롤러만 이용하는 경우
<br/>컨테이너에 들어가는 애플리케이션의 소스가 변경된경우 다시 레플리케이션 컨트롤러를 새로 만들고 rolling-update 를 수행했다.
<br/>
> 참고) 롤링 업데이트는 파드 인스턴스를 점진적으로 새로운 것으로 업데이트하여 디플로이먼트 업데이트가 서비스 중단 없이 이루어질 수 있도록 해준다.
<br/> https://kubernetes.io/ko/docs/tutorials/kubernetes-basics/update/update-intro/
<br/><br/>

그러나 deployment가 등장하며 pod의 컨테이너의 이미지만 변경해주면 편리하게 업데이트가 되며
<br/>히스토리 확인 및 롤백기능까지 사용할 수 있게 되었다.
<br/>
###2. **yaml파일을 이용해 생성하기**
<br/>apiVersion: apps/v1
<br/>kind: Deployment
<br/>metadata:
<br/>  name: mydeploy
<br/>spec:
<br/>  replicas: 3
<br/>  selector:
<br/>    matchLabels:
<br/>      app: myHelloWorld
<br/>  template:
<br/>    metadata:
<br/>      labels:
<br/>        app: myHelloWorld
<br/>    spec:
<br/>      containers:
<br/>      - name: myapp
<br/>        image: repo/helloworld:1
<br/>deployment의 yaml파일은 저번 게시글의 replicaset과 kind빼고는 모두 똑같다.
<br/>
<br/>3. 동작확인
<br/>$ kubectl apply -f deployment1.yaml --record
<br/>deployment.apps/mydeploy created
<br/>yaml파일을 적용해 deployment를 생성한다.
<br/>deployment는 --record를 적용해줍니다.(추후 히스토리 확인용)
<br/>
<br/>$ kubectl get all
<br/>NAME                            READY   STATUS    RESTARTS   AGE
<br/>pod/mydeploy-5bd587868d-8442c   1/1     Running   0          85s
<br/>pod/mydeploy-5bd587868d-dgknp   1/1     Running   0          85s
<br/>pod/mydeploy-5bd587868d-txg9s   1/1     Running   0          85s
<br/>
<br/>NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
<br/>service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   8d
<br/>
<br/>NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
<br/>deployment.apps/mydeploy   3/3     3            3           85s
<br/>
<br/>NAME                                  DESIRED   CURRENT   READY   AGE
<br/>replicaset.apps/mydeploy-5bd587868d   3         3         3       85s
<br/>kubectl get all로 모두 확인해보니 여러 resource들이 올라와있다.
<br/>deployment는 replicaset을 생성하고
<br/>replicaset은 pod을 생성하여 생긴 결과이다.
<br/>
<br/>
### 4. **이미지 업데이트하기**
<pre>
<br/>@RestController
<br/>public class DemoController {
<br/>    @GetMapping("/")
<br/>    public String test() {
<br/>        return "hello new world";
<br/>    }
<br/>}
</pre>
<br/>hello world에서 hello new world를 리턴해주는 웹 애플리케이션으로 이미지가 업데이트 된 경우이다.
<br/>helloworld:2에 이미지를 올려주고 deployment yaml파일에 가서
<br/>
<br/>apiVersion: apps/v1
<br/>kind: Deployment
<br/>metadata:
<br/>  name: mydeploy
<br/>spec:
<br/>  replicas: 3
<br/>  selector:
<br/>    matchLabels:
<br/>      app: myHelloWorld
<br/>  template:
<br/>    metadata:
<br/>      labels:
<br/>        app: myHelloWorld
<br/>    spec:
<br/>      containers:
<br/>      - name: myapp
<br/>        image: repo/helloworld:2
<br/>        # pod의 image만 변경
<br/>image에 태그만 변경해 주고 apply 한다.
<br/>$ kubectl apply -f deployment2.yaml --record
<br/>deployment.apps/mydeploy configured
<br/>deployment는 --record 를 붙여줘야 history에서 확인이 가능하다.
<br/>
<br/>
### 5. **deployment rollout 이용하기**
<br/>$ kubectl rollout status deployment mydeploy
<br/>deployment "mydeploy" successfully rolled out
<br/>먼저 rollout status 를 확인한다.
<br/>rollout은 여러개의 pod를 모두 죽이지않고 순차적으로 업데이트하는 방식을 말한다.
<br/>
<br/>$ kubectl rollout history deployment mydeploy
<br/>REVISION  CHANGE-CAUSE
<br/>2         kubectl.exe apply --filename=mydeployment.yaml --record=true
<br/>3         kubectl.exe apply --filename=mydeployment.yaml --record=true
<br/>--record 옵션을 붙여서 apply 한 deployment의 경우 revision이 찍힌다.
<br/>Revision 번호를 확인했으면 해당 revision을 확인해본다.
<br/>
<br/>$ kubectl rollout history deployment mydeploy --revision=2
<br/>deployment.apps/mydeploy with revision #2
<br/>Pod Template:
<br/>  Labels:       app=myHelloWorld
<br/>        pod-template-hash=5bd587868d
<br/>  Annotations:  kubernetes.io/change-cause: kubectl.exe apply --filename=mydeployment.yaml --record=true
<br/>  Containers:
<br/>   myapp:
<br/>    Image:      repo/helloworld:1
<br/>    Port:       <none>
<br/>    Host Port:  <none>
<br/>    Environment:        <none>
<br/>    Mounts:     <none>
<br/>  Volumes:      <none>
<br/>이전 버전이므로 revision3의 이미지태그가 1인걸 확인할 수 있다.
<br/>
<br/>해당 이미지로 롤백하고싶다면
<br/>yaml파일을 수정해서 다시 apply해도 되지만, 변경된부분이 많다면 rollout undo명령어를 활용해도 된다.
<br/>
<br/>$ kubectl rollout undo deployment mydeploy --to-revision=1
<br/>deployment.apps/mydeploy rolled back
<br/>pod, replicaset, deployment를 알면 컨테이너를 생성, 복제, 유지하고 히스토리를 확인해서 롤백할 수 있다.