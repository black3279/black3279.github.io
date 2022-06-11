---
layout: post
author: Bruce Lee
title: Kubernetes 레플리카셋
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

