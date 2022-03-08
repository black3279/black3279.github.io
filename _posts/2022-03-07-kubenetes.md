---
layout: post
author: Bruce Lee
title: Kubenetes
---
👨‍🎓 쿠버네티스 입문 !

🐧 Kubeadm 이란 쿠버네티스에서 공식 제공하는 클러스터 생성/관리 도구이다. 여러 대 서버를 쿠네티스 클러스터로 손쉽게 구성할 수 있다.
<br/>(Image from 쿠버네티스 입문)![Untitled](/assets/img/Kubenetes/img01.PNG)<br/>
- 여러 대의 마스터 노드를 구성하고 그 앞에 로드밸런서를 두었으며 워커 노드들이 마스터 노드에 접근할 때는 로드밸런서를 거쳐 접근한다
- 마스터 노드 1대에 장애가 발생하더라도 노드밸런서에서 다른 마스터 노드로 접근할 수 있게 해서 클러스터의 신뢰성을 유지한다.

🎡 Kubectl 이란 쿠버네티스 클러스터를 관리하는 커맨드라인 인터페이스이다. kubectl 에서는 쿠버스네티스 자원들의 생성, 업데이트, 삭제, 디버그, 모니터링, 트러블슈팅, 클러스터 관리에 대한 명령어를 지원한다.
- 테스트를 위해서 쿠버네티스의 파드들에 접근할 때 필요한 echo 라는 server 서비스를 생성한다.
```dockerfile
kubectl run echo --image=gcr.io/google_containers/echoserver:1.8 --port=8080
```
<br/>![Untitled](/assets/img/Kubenetes/img02.PNG)<br/>
- 다음으로 에코 서버에 접근할 수 있도록 로컬 컴퓨터로 포트포워딩을 해보자.
> 포트포워딩 이란 ?
포트 포워딩이란 컴퓨터 네트워크에서 패킷이 라우터나 방화벽 같은 네트워크 게이트웨이를 통과하는 동안 네트워크 주소를 변환해주는 것을 의미한다. 쉽게 말해 외부에서 접속이 가능하도록 하는 것이다.
<br/>![Untitled](/assets/img/Kubenetes/img03.PNG)<br/>
### 결과확인
```
Hostname: echo

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=127.0.0.1
	method=GET
	real path=/
	query=
	request_version=1.1
	request_uri=http://localhost:8080/

Request Headers:
	accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
	accept-encoding=gzip, deflate, br
	accept-language=ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
	connection=keep-alive
	cookie=voter_id=ea4617c26f048e34; io=ZiH0cWnOMUqXnm7wAAAA
	host=localhost:8080
	sec-ch-ua=&quot; Not A;Brand&quot;;v=&quot;99&quot;, &quot;Chromium&quot;;v=&quot;99&quot;, &quot;Google Chrome&quot;;v=&quot;99&quot;
	sec-ch-ua-mobile=?0
	sec-ch-ua-platform=&quot;Windows&quot;
	sec-fetch-dest=document
	sec-fetch-mode=navigate
	sec-fetch-site=none
	sec-fetch-user=?1
	upgrade-insecure-requests=1
	user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.51 Safari/537.36

Request Body:
	-no body in request-
```
- 이후 에코 서버의 실행 중 로그를 수집할 때는 kubectl logs -f 파드이름 을 실행한다.

```dockerfile
> kubectl logs -f echo
127.0.0.1 - - [08/Mar/2022:13:38:04 +0000] "GET / HTTP/1.1" 200 1110 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.51 Safari/537.36"
127.0.0.1 - - [08/Mar/2022:13:38:04 +0000] "GET /favicon.ico HTTP/1.1" 200 1048 "http://localhost:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.51 Safari/537.36"
127.0.0.1 - - [08/Mar/2022:14:09:19 +0000] "GET / HTTP/1.1" 200 470 "-" "Mozilla/5.0 (Windows NT; Windows NT 10.0; ko-KR) WindowsPowerShell/5.1.19041.1320"
```
- 해당과 같이 시간, HTTP 버전, 웹 브라우저와 운영체제 버전 등의 정보를 확인할 수 있다.
- 마지막으로 실습한 파드와 서비스를 삭제해보자
- 먼저 쉘에서 에코 서버 로그 수집과 에코 서버 실행을 중지한다.
```dockerfile
PS C:\Users\30133\Desktop\Docker\example-voting-app-master> kubectl delete pod echo
pod "echo" deleted
PS C:\Users\30133\Desktop\Docker\example-voting-app-master> kubectl get pods
No resources found in default namespace.
PS C:\Users\30133\Desktop\Docker\example-voting-app-master> kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   81m
```
- 정상 삭제된 것을 확인할 수 있다.