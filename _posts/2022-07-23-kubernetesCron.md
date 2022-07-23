---
layout: post
author: Bruce Lee
title: Kubernetes 크론
---

## 👨‍🎓 1. cronjob이란<br/>
- 리눅스 crontab(작업 예약 스케줄러) 쿠버네티스 버전이다.<br/>
  <br/>
-  백업, 소나큐브 분석, 이메일 전송 등 정기적이고 반복적인 작업을 하는데 쓰이다.<br/>
  <br/>
  <br/>
  <br/>
### 2.  사용법 <br/>
(1). mycj.yaml<br/>
<br/>

> apiVersion: batch/v1<br/>
kind: CronJob<br/>
metadata:<br/>
name: time-limited-cronjob<br/>
spec:<br/>
jobTemplate:<br/>
metadata:<br/>
name: my-time-limited-job<br/>
spec:<br/>
activeDeadlineSeconds: 150 # 150초만 실행하고 끝내는 예제<br/>
template:<br/>
spec:<br/>
containers:<br/>
> - args:<br/>
> - /bin/sh<br/>
> - -c<br/>
> - date; echo Hello CronJob<br/>
image: busybox<br/>
name: time-limited-job<br/>
restartPolicy: Never<br/>
schedule: '* * * * *'<br/>

- yaml파일을 정의합니다.<br/>
<br/>
- schedule에 있는 * * * * *는 우리가 알고있는 리눅스 crontab 방식으로 사용하면 된다.<br/>
<br/>
- 나는 현재시간과, Hello Cronjob을 출력해주는 이미지를 사용하겠다.<br/>
<br/>
### (2). 실행 및 결과확인<br/>
<br/>
- cronjob을 만들어준다.<br/>
<br/>

> $ kubectl apply -f mycj.yaml <br/>
cronjob.batch/time-limited-cronjob created<br/>
<br/>

- cronjob 확인<br/>
<br/>
> $ kubectl get cj<br/>
NAME                   SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE<br/>
time-limited-cronjob   * * * * *   False     0        49s             15m<br/>
<br/>
<br/>

- 크론잡은 deployment - replicaset - pod 과 비슷한 특성이 있다.<br/>
- cronjob은 job을 만들고 job은 pod을 생성합니다.(이름과 해시로 확인)<br/>
- cronjob은 1분에 한번씩 job을 생성하고(* * * * * 이므로 1분에 한번씩) job은 pod을 생성하게 된다.<br/>
<br/>
> $ kubectl get all<br/>
NAME                                      READY   STATUS      RESTARTS   AGE<br/>
pod/time-limited-cronjob-27144410-sgzzx   0/1     Completed   0          2m29s<br/>
pod/time-limited-cronjob-27144411-w4pl7   0/1     Completed   0          89s<br/>
pod/time-limited-cronjob-27144412-c9qqf   0/1     Completed   0          29s<br/>
<br/>
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE<br/>
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   15m<br/>
<br/>
NAME                                 SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE<br/>
cronjob.batch/time-limited-cronjob   * * * * *   False     0        29s             14m<br/>
<br/>
NAME                                      COMPLETIONS   DURATION   AGE<br/>
job.batch/time-limited-cronjob-27144410   1/1           3s         2m29s<br/>
job.batch/time-limited-cronjob-27144411   1/1           4s         89s<br/>
job.batch/time-limited-cronjob-27144412   1/1           3s         29s<br/>
<br/>
<br/>

- pod의 로그를 찍어 정상적으로 job이 실행된것을 확인할 수 있다.<br/>
<br/>

> $ kubectl logs time-limited-cronjob-27144412-c9qqf<br/>
Wed Aug 11 06:52:03 UTC 2021<br/>
Hello CronJob