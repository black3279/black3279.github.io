---
layout: post
author: Bruce Lee
title: Reverse Proxy with Docker, GA and lint-staged Issue
---

## 👨‍🎓 1. Reverse Proxy 란<br/><br/>
- 클라이언트 요청을 대신 받아 내부 서버로 전달해주는 것을 리버스 프록시(Reverse Proxy) 라고 한다.<br/>
- 프록시란 대리라는 의미로, 정보를 대신 전달해주는 주체라고 생각하면 되는데, 만약 이 프록시 없이 웹 서버를 운영한다고 가정한다.<br/>
- 웹서버를 열어서 운영했을 때, 사용자가 갑자기 많아지거나, 웹서버가 그대로 노출되어 있기 때문에 보안적으로 위험성이 있다. nginx를 사용하면 로드 밸런싱으로 부하를 줄여줄 수 있고, 분산 처리 또한 가능하며 웹서버의 SSL 인증도 적용할 수 있다.<br/>
- 따라서 사용자 -> nginx -> 웹서버로 구성해서 사용자의 요청을 nginx가 대신 웹서버로 전달해주도록 구성한다.<br/>
  <br/>
2. 리버스 프록시의 장점<br/>
   <br/>
* 장점은 아래와 같다.<br/>
  <br/>
- 로드 밸런싱 : Nginx는 클라이언트의 요청을 프록시 서버에 분산하기 위해 로드 밸런싱을 수행하여 성능, 확장성 및 신뢰성을 향상시킬 수 있다.<br/>
- 캐싱 : Nginx를 역방향 프록시로 사용하면 미리 렌더링된 버전의 페이지를 캐시하여 페이지 로드 시간을 단축할 수 있다. 이 기능은 프록시 서버의 응답에서 수신한 콘텐츠를 캐싱하고 이 콘텐츠를 사용하여 매번 동일한 콘텐츠를 프록시 서버에 연결할 필요 없이 클라이언트에 응답하는 방식으로 작동한다.<br/>
- SSL 터미네이션 : Nginx는 클라이언트와의 연결에 대한 SSL 끝점 역할을 할 수 있다. 수신 SSL 연결을 처리 및 해독하고 프록시 서버의 응답을 암호화한다.<br/>
- 압축 : 프록시 서버가 압축된 응답을 보내지 않는 경우 클라이언트로 보내기 전에 응답을 압축하도록 Nginx를 구성할 수 있다.<br/>
- DDoS 공격 완화 : 수신 요청과 단일 IP 주소당 연결 수를 일반 사용자에게 일반적인 값으로 제한할 수 있다. 또한 Nginx를 사용하면 클라이언트 위치와 "User-에이전트" 및 "Referer"와 같은 요청 헤더 값을 기준으로 액세스를 차단하거나 제한할 수 있다.<br/>
  <br/>
3. 코드<br/>
   <br/>
   nginx.conf<br/>
   <br/>

``` bash
log_format upstream_time '$remote_addr - $remote_user $ request'

server {
    listen  443 ssl;
    listen [::]:443 ssl;
    resolver 8.8.8.8l
	
    ssl_certificate     /etc/nginx/conf.d/cert;
    ssl_certificate_key /etc/nginx/conf.d/key;

    access_log  /var/log/nginx/host.access.log upstream_time;
    error_log   /var/log/nginx/host.error.log;
    
    location / {
        proxy_pass         https://next-server$request_uri;
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
    }
}
```

- 주의깊게 봐야할 부분은 http 블럭 안의 upstream 블럭과 server 블럭이다.<br/>
  1. 사용자가 nginx 서버 주소로 요청한다(localhost:443). 기본적으로 http는 80포트를 사용하기 때문에 nginx에서는 listen 80포트로 구성한다.<br/>
     <br/>
  2. nginx서버는 location 블럭에서 proxy_pass로 지정된 주소로 요청을 전달해준다.<br/>
     <br/>
  3. 만약 운영중인 서버가 localhost:8080, 혹은 localhost:3000이면 upstream 블럭에 localhost:8080, localhost:3000으로 적어준다.<br/>
     <br/>
  4. 웹서버(port 3000)를 시작하고, nginx도 시작해준다.<br/>
     <br/>
  5. nginx 서버 주소로 요청하면(localhost:443), proxy_pass로 지정한 웹 서버로 요청이 전달된다.<br/>
6. docker-compose 사용시<br/>
   docker-compose.yml<br/>
   <br/>

``` bash
version: "1.0"

services:
    api:
        image: nginx
        ports:
            - "443:443"
        volumes:
            - ./conf.d:/etc/nginx/conf.d/
```

* 유용한 명령어

```bash
ssh 계정@ip -p port
scp -P 포트 cert 계정@ip:/dir
sudo -i

yum -y update
yum-config-manager --add-repo docker-ce.repo
yum install -y docker-ce

systemctl enable --now docker
systemctl status docker
docker -v

yum install docker-compose-plugin

docker compose up --detach
docker exec -it hashnumber /bin/bash
```

### 👨‍🎓 2. Global Accelerator 란<br/><br/>
- Global Accelerator(글로벌 액셀러레이터)는 AWS의 글로벌 네트워크 인프라를 통해 사용자 트래픽을 전송하여 인터넷 사용자 성능을 최대 60% 개선하는 네트워킹 서비스이다.<br/>
- Global Accelerator는 사용자와 가장 가까운 위치의 사용 가능한 정상 엔드포인트로 트래픽을 자동으로 재라우팅하여 엔드포인트 장애를 완화한다. Global Accelerator의 자동 라우팅 최적화 기능은 인터넷이 혼잡할 때 패킷 손실, 지터 및 지연 시간을 일관적으로 낮게 유지한다.<br/>
- Global Accelerator는 2개의 글로벌 정적 고객용 IP를 제공한다. 트래픽이 도착하는 최종 엔드포인트는 AWS 특정 리전의 ELB(Network Load Balancer 혹은 Application Load Balancer), 탄력적 IP 및 EC2 인스턴스이며, 이를 AWS 애플리케이션 오리진으로 구성하는 방식이다.<br/>
  <br/>

* 시나리오1: 단일 리전 내에 존재하는 앱의 성능 개선<br/>
- 전 세계의 엣지 로케이션(고정 진입점 역할)에 글로벌 정적 애니캐스트 IP 2개(아래 사진 기준: 4.3.2.1, 9.8.7.6) 부여하여 실제 사용자는 이 주소를 통해 자신의 인근 엣지에 접속<br/>
- 특정 리전에 존재하는 엔드 포인트에 전 세계 사용자가 빠르게 접속<br/>
  <br/>(Image from https://velog.io/@khyup0629/AWS-Global-Accelerator)![Untitled](/assets/img/Reverse_Proxy/case1.png)<br/>
  <br/>

* 시나리오 2: 다중 리전 내에 존재하는 앱의 성능 개선<br/>
- 전 세계의 엣지 로케이션(고정 진입점 역할)에 글로벌 정적 애니캐스트 IP 2개(아래 사진 기준: 4.3.2.1, 9.8.7.6) 부여하여 실제 사용자는 이 주소를 통해 자신의 인근 엣지에 접속<br/>
- 최대 10개 AWS 리전으로 접속 가능<br/>
  <br/>(Image from https://velog.io/@khyup0629/AWS-Global-Accelerator)![Untitled](/assets/img/Reverse_Proxy/case2.png)<br/>
  <br/>
### * 장점 (AWS 전용선 사용)<br/>
1. 성능 개선<br/>
   - 사용자와 가장 가까운 위치의 사용 가능한 정상 엔드포인트로 트래픽을 자동으로 라우팅하여 접속 속도 개선.<br/>

2.즉각적 장애 조치<br/>
   - 애플리케이션 장애 시 AWS Global Accelerator는 차선의 엔드포인트로 즉각적인 장애 조치를 수행.<br/>

3. 네트워크 영역을 활용한 내결함성<br/>
   - AWS Global Accelerator는 독립적 네트워크 영역에서 서비스되는 2개의 정적 IPv4 주소를 할당한다. 가용 영역과 유사하게, 이러한 네트워크 영역은 자체 물리적 인프라를 갖춘 격리된 유닛으로서 고유한 IP 서브넷에서 IP 주소를 서비스한다.<br/>
   - 네트워크 영역의 IP 주소 1개가 특정 클라이언트 네트워크에 의한 IP 주소 차단 또는 네트워크 중단으로 인해 사용할 수 없게 되는 경우, 클라이언트 애플리케이션은 다른 격리 네트워크 영역에서 정상적인 정적 IP 주소를 사용하여 재시도할 수 있다.<br/>

### 따라서, AWS EC2 에 도커를 사용한 nginx 서버를 여러대 구성한 후, alb 를 통해 부하 분산 및 Fail Over 가 가능하다.<br/>
### 또한, 거기에 GA 를 추가하여 리전별 접근 속도를 높이거나 고정 ip 를 사용할 수 있다.<br/>
### NLB 와 ALB 에 따라서 AWS 전용 네트워크 사용 여부가 다를 수 있으므로 해당을 주의하여 설계한다.<br/>

### 👨‍🎓 3. Lint-staged Issue<br/><br/>
- https://www.npmjs.com/package/lint-staged<br/>
- 해당에 따르면 Since v12.0.0 lint-staged is a pure ESM module, so make sure your Node.js version is at least 12.20.0, 14.13.1, or 16.0.0. Read more about ESM modules from the official Node.js<br/>
- 즉, 12 버전을 쓸 경우 노드 12.20 을 사용하여도 문제가 없었다.<br/>

* 문제가 되었던 부분 <br/>
- package.json<br/>

``` javascript
"supports-color": "^9.0.2",
```

- supports-color 의 최신 버전을 보자 (가장 최신은 패치됨)<br/>
- https://github.com/chalk/supports-color<br/>

``` javascript
/// function hasFlag(flag, argv = globalThis.Deno?.args ?? process.argv) {
function hasFlag(flag, argv = globalThis.Deno ? globalThis.Deno.args : process.argv) {
	const prefix = flag.startsWith('-') ? '' : (flag.length === 1 ? '-' : '--');
	const position = argv.indexOf(prefix + flag);
	const terminatorPosition = argv.indexOf('--');
	return position !== -1 && (terminatorPosition === -1 || position < terminatorPosition);
}
```

- 즉, Node v14.0.0 에 호환되는 ES2020 문법인 Optional Chainning 을 사용함으로써 해당 라이브러리를 사용하는 모든 패키지의 노드 호환성을 파괴해버린 것...<br/>
- 관련 PR<br/>
- https://github.com/chalk/supports-color/commit/74a0d5a0f8e6ec23ea17bbc037a778510012e80d
<br/>
- https://github.com/okonet/lint-staged/commit/50f95b3d51e69074ab5ff5ddb7147828fcd85b7b
<br/>
