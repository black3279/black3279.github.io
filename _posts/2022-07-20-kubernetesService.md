---
layout: post
author: Bruce Lee
title: Kubernetes 서비스
---

## 👨‍🎓 1. service란<br/>
- pod은 자체 ip를 가지고있으며 같은 클러스터 내부라면 해당 ip를 통해 통신을 할 수 있다.<br/>
- 그러나 pod의 ip는 고정 되어 있지 않다.(pod에 문제가 생겨 replicaset이 다시 생성해주면 ip가 변경되게 된다)<br/>
- ip를 이용해 통신하면 pod에 문제가 생겨 죽고 다시 살아났을때 더이상 통신이 안된다.<br/>
- 이를 방지하기 위해 service(cluster ip)를 사용한다.<br/>
### 2. deployment를 생성하여 pod을 생성<br/>

>   apiVersion: apps/v1<br/>
   kind: Deployment<br/>
   metadata:<br/>
   name: mydeploy<br/>
   spec:<br/>
   replicas: 2<br/>
   selector:<br/>
   matchLabels:<br/>
   app: myHelloWorld<br/>
   template:<br/>
   metadata:<br/>
   labels:<br/>
   app: myHelloWorld<br/>
   spec:<br/>
   containers:<br/>
>   - name: myapp<br/>
     image: repo/helloworld:1<br/>
     ports:<br/>
      - containerPort: 8080<br/>
        protocol: TCP<br/>
        deployment 에서 사용한 deployment를 재활용한다.<br/>
<br/>

### 3. service(clusterip) 생성<br/>
- kubectl expose deployment mydeployment라는 명령어만 쳐도 편하게 servcie를 생성할 수 있지만<br/> 보통 yaml파일로 정의하여 생성한다.<br/>
<br/>

> apiVersion: v1<br/>
kind: Service<br/>
metadata:<br/>
name: mySvc<br/>
spec:<br/>
type: ClusterIP # 적지 않아도 service의 spec의 type은 default가 clusterip이다.<br/>
clusterIP: 10.102.217.210 # 적지 않아도 10으로 시작되는 ip를 할당해준다.<br/>
ports:<br/>
<br/>

- port: 80 # 서비스가 사용할 포트<br/>
- targetPort: 8080 #서비스가 포워드할 컨테이너포트<br/>
- selector:<br/>
  - app: myHelloWorld # app=MyHelloworld인 모든 pod는 이 서비스에 속한다.<br/>clusterip의 80번포트를 app: myHelloWorld라는 라벨이 있는 팟에 8080포트로 포워딩 시켜주겠다는 의미이다.<br/>
  <br/> 컨테이너내부의 애플리케이션의 포트와 targetPort를 동일하게 해줘야 한다.<br/>

### 4. service 생성 확인<br/>


> $ kubectl get svc
   NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
   kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP   15m
   myclusterip   ClusterIP   10.102.217.210   <none>        80/TCP    11m

<br/>* myclusterip가 생성된걸 확인할 수 있다.<br/>

> $ kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
mydeploy-599887685-hc9tn   1/1     Running   0          2m14s   172.17.0.3   minikube   <none>           <none>
mydeploy-599887685-kqk4n   1/1     Running   0          2m14s   172.17.0.2   minikube   <none>           <none>

<br/>
- kubectl get pod -o wide 명령어를 통해 pod의 ip를 확인할수 있지만 해당 ip는 pod이 죽고 다시 생성되면 변하므로 단순히 테스트하는경우에만 사용한다. pod ip(172.17.0.2 or 172.17.0.3)의 8080은<br/>
<br/>
service(clusterip)의 ip(10.102.217.210)의 80번으로 연결되게 된다.<br/>

> $ kubectl get ep<br/>
NAME          ENDPOINTS                         AGE<br/>
kubernetes    192.168.49.2:8443                 3d<br/>
myclusterip   172.17.0.2:8080,172.17.0.4:8080   3d<br/>
<br/>
<br/>

- endpoint를 확인할 수도 있다.<br/>
- 내 쿠버네티스의 endpoint를 확인해보니 팟들이 노출된것도 확인할 수 있다.<br/>

### 5. 동작확인<br/>
 
>  $ kubectl run test --image=busybox -it --rm -- sh<br/>
<br/>
/ # wget 10.102.217.210:80<br/>
Connecting to 10.102.217.210:80 (10.102.217.210:80)<br/>
<br/>
saving to 'index.html'<br/>
index.html           100% |**************************************|    11  0:00:00 ETA<br/>
'index.html' saved<br/>
<br/>
/ # cat index.html<br/>
hello world<br/>
/ # exit<br/>
<br/>

- busybox 이미지를 이용하여 pod을 하나 만든후<br/>
- 생성했던 클러스터ip의 80번 포트에 wget 요청을 한다.<br/>
- hello world가 정상적으로 출력되는것을 확인할 수 있다.<br/>
### 6. clusterip만 있으면 누구든 pod에 접근할수 있을까?<br/>
- 이제 pod과 pod은 어떤 노드(서버)에 있든지간에 통신이 가능해졌다.<br/>
- 프론트엔드와 백엔드와 디비를 세대의 서버에 띄워놓고 통신 시킬 수 있게 됐다.<br/>
- 그러나 clusterip는 클러스터 내부에서만 통신 할 수 있다.<br/>
- 외부에 통신을 하려면 nodeport나 loadbalencer를 만들어 줘야한다.<br/>
- 다음에서 nodeport에 대해 알아보겠다.

## 👨‍🎓 1. NodePort란<br/>
- ClusterIP를 통해서 어떤 노드(서버)에 있더라도 pod과 pod끼리 통신을 시킬수 있었다.<br/>
- 즉 api서버와 db서버를 통신시킬수 있게 됐다.<br/>
- 그러나 clusterIP의 특성상 하나의 쿠버네티스 클러스터 안에서만 통신이 가능하다는 단점이 있었다.<br/>
- 이제 pod을 외부에 공개시킬 순서이다.<br/>
  <br/>
### 2. deployment를 생성하여 pod를 생성<br/>
- deployment 에서 사용한 deployment를 재활용한다.<br/>
<br/>
### 3. service(nodePort) 생성<br/>
<br/>
>   apiVersion: v1<br/>
   kind: Service<br/>
   metadata:<br/>
   name: mynp<br/>
   spec:<br/>
   type: NodePort # type을 적지않으면 default는 clusterip이므로 꼭 적어준다.<br/>
   ports:<br/>
>  port: 80 # 서비스가 사용할 포트<br/>
  targetPort: 8080 #서비스가 포워드할 컨테이너포트<br/>
  nodePort: 31212 # 서비스의 노드포트(적지않으면 30000~32768중 자동할당)<br/>
  selector:<br/>

- app: myHelloWorld # app=MyHelloworld인 모든 pod는 이 서비스에 속한다.<br/>
- clusterip와 다른점은 type을 nodeport로 해준점(default는 ClusterIP)이다.<br/>
- nodeport를 설정해주지않으면 3만번대에서 자동 할당하지만 중요한 부분이니 적었다.<br/>
<br/>
- 이 서비스를 클러스터에 올리면 외부(웹브라우저)에서 쿠버네티스 ip:31212 에 접근하면 pod에 접근할 수 있다.<br/>
<br/>
### 4. 동작확인<br/>
-   nodeport를 생성해준다.<br/>
<br/>

>$ kubectl apply -f mynodeport.yaml<br/>
service/mynp created<br/>
<br/>

- service를 확인한다.<br/>
<br/>
> $ kubectl get svc<br/>
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE<br/>
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        3d<br/>
mynp         NodePort    10.109.53.237   <none>        80:31212/TCP   8s<br/>

- type이 nodeport인 resource가 하나 올라와있다.<br/>
- 노드포트는 클러스터 ip의 기능을 포함한다.<br/>
- 이제 쿠버네티스 클러스터에 할당된 ip의 31212포트에 접근하면 pod에 접근할 수 있다.<br/>
<br/>

> $ kubectl get ep<br/>
NAME         ENDPOINTS                         AGE<br/>
kubernetes   192.168.49.2:8443                 3d<br/>
mynp         172.17.0.2:8080,172.17.0.4:8080   111s<br/>
<br/>
$ minikube service mynp<br/>
|-----------|------|-------------|---------------------------|<br/>
| NAMESPACE | NAME | TARGET PORT |            URL            |<br/>
|-----------|------|-------------|---------------------------|<br/>
| default   | mynp |          80 | http://192.168.49.2:31212 |<br/>
|-----------|------|-------------|---------------------------|<br/>
🏃  mynp 서비스의 터널을 시작하는 중<br/>
|-----------|------|-------------|------------------------|<br/>
| NAMESPACE | NAME | TARGET PORT |          URL           |<br/>
|-----------|------|-------------|------------------------|<br/>
| default   | mynp |             | http://127.0.0.1:58836 |<br/>
|-----------|------|-------------|------------------------|<br/>
🎉  Opening service default/mynp in default browser...<br/>
❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.<br/>
<br/>
<br/>

- kubectl get ep를 입력해 endpoint를 확인하면 쿠버네티스의 ip를 확인할 수 있다.<br/>
- 실제로 pc에 랜처, kubeadm등을 이용해 쿠버네티스를 깔았고, 공유기를 열고 포트프록시 설정을 해줬다면 쿠버네티스 ip를 진짜 외부에서 접근할 수 있겠지만<br/>
- 예제를 위한 클러스터인 minikube나 wsl2를 사용하고있으므로 웹 브라우저에서 접근하려면 다른 복잡한 방법을 사용해야한다.<br/>
- 대신 minikube에 좋은 명령어가 있다.<br/>minikube service [nodeport이름]<br/>
  - 해당명령어를 입력하면 192.168.49.2:31212를 자동으로 프록시해서 접근해준다.<br/>
  <br/>
- 내 웹 브라우저에서 내 웹 애플리케이션에 성공적으로 접근한것을 확인 할 수 있다.