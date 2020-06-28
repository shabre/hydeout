---
layout: post
title: kubernetes 에서 코드와 상호작용
categories: kubernetes
excerpt_separator: "<!--more-->"
---

## 컨테이너 이미지 빌드를 위한 실용적인 팁

- 소스코드 저장소에 도커파일을 함께 보관하라. 이렇게 할 경우 COPY, ADD 옵션을 사용해 소스코드가 위치한 폴더를 기준으로 상대경로에 있는 파일들을 참조할 수 있다.
- 컨테이너 이미지를 만들기 위한 별도의 스크립트 파일을 작성해 관리하라. 개발 언어 및 프레임워크에 구애받지 않게 자동화 프로세스를 만드는데 도움이 된다.(cf 코드작성, 컴파일, 테스트, 유효성 검사 프로세스와 함께 컨테이너 이미지를 생성)
- 베이스 이미지에 디버깅이나 점검할 수 있는 추가 도구를 설치하라. 단, 최소한의 도구만을 설치하라.(불필요한 이미지 사이즈, 해커에의한 악용 방지)
- 컨테이너 이미지 빌드 시 프로덕션 레벨에서 사용될 수 있음을 염두하고 기본값으로 빌드하라.(개발관련 디펜던시가 포함되지 않도록)
- 생성한 모든 컨테이너를 읽기 전용으로 설정하라. 만약 로컬 파일에 대한 쓰기 권한이 필요할 경우 컨테이너 내부에 파일이 저장될 위치를 명시해야한다.

### 프로그램의 결과 전송

kubectl logs 실행 시 stdout, stderr과 컨테이너에서 발생하는 모든 로그를 출력하도록 되어있다. 컨테이너 내부에서 생성되는 로그를 로컬파일에 저장 시 불필요한 디스크 I/O가 발생한다. 따라서 로그 수집기를 추가할 경우 어플리케이션이 배포된 컨테이너나 팟 외부에 로그를 수집, 전송, 처리 기능을 구현하는것이 일반적이다.

## 로그

### 하나 이상의 컨테이너로 구성된 팟

로그를 확인하고자 하는 경우, 만약 팟이 하나의 컨테이너로만 구성돼 있을 경우 컨테이너를 지정할 필요가 없다. 하지만 팟이 두개 이상일 경우엔 -c 옵션을 추가하거나 logs 커맨드 뒤에 특정 컨테이너를 추가해 지정할 수 있다.

만약 flask와 background로 구성된 webapp 팟에서 background 컨테이너에서 발생하는 로그를 확인하고 싶으면
`kubectl logs webapp background` 혹은 `kubectl logs webapp -c background` 커맨드를 사용해 확인이 가능하다.

디플로이먼트에 포함된 팟과 컨테이너의 로그를 확인할 때는 팟에 자동으로 할당된 풀네임을 사용하는 대신 디플로이먼트의 팟 이름의 프리픽스만 사용하여 확인할 수 있다.

```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl logs deployment/flask
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 215-423-890
 ```

### 로그 스트리밍

 -f 옵션을 추가하면 지속적으로 발생하는 로그를 확인할 수 있다. 지속적으로 로그를 출력하기 때문에 포그라운드 상태로 프로세스가 실행된다.
 ```
 Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl logs deployment/flask -f
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 215-423-890
 ```

### 이전 로그 확인

컨테이너에 오류가 발생하거나 업데이트 수행 결과 정상적으로 동작하지 않은 경우의 로그를 확인하려면 -p 옵션을 주면 확인이 가능하다.

### 타임스탬프
로그 출력 시의 타임스탬프도 같이 출력할 수 있다.
--timestamps 옵션을 추가하면 출력 가능하다.
```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl logs deployment/flask --timestamps
2020-06-27T04:28:53.021962166Z  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
2020-06-27T04:28:53.023187291Z  * Restarting with stat
2020-06-27T04:28:53.286647049Z  * Debugger is active!
2020-06-27T04:28:53.305084993Z  * Debugger PIN: 215-423-890
```

## 디버깅 기법

디버깅 기법은 크게 세가지가 있다.
- 대화형으로 이미지 배포하기
- 실행 중인 팟에 접속하기
- 컨테이너에 두번째 프로세스 실행하기

### 대화형으로 이미지 배포하기

대화형으로 실행하면 아래와 같이 / # 에 커맨드를 입력할 수 있도록 노출된다.
```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl run -i -t alpine-interactive --image=alpine -- sh
If you don't see a command prompt, try pressing enter.
/ # 
```
-i: 대화형 세션 생성
-t: 대화형 세션의 출력을 내보내기 위한 tty 세션 할당
--sh: 세션 실행 시 특정 커맨들을 실행할 수 있도록 하는 옵션. 위 예제에서는 셸 실행.

ctrl + D 나 exit를 입력할 경우 셸을 빠져나올 수 있다.

이렇게 셸을 빠져나올 경우 디플로이먼트는 여전히 실행되고 있기 때문에 kubectl attach 커맨드를 이용해 다시 접속할 수 있다.

```
//확인
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
alpine-interactive       1/1     Running   1          10m
flask                    1/1     Running   2          5d16h
flask-665cf87fc4-pbmzg   1/1     Running   2          5d15h

//디버깅 세션 다시열기
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl attach alpine-interactive -i -t
Defaulting container name to alpine-interactive.
Use 'kubectl describe pod/alpine-interactive -n default' to see all of the containers in this pod.
If you don't see a command prompt, try pressing enter.
/ # 

//팟 삭제
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl delete pods alpine-interactive
pod "alpine-interactive" deleted
```

### 실행중인 팟에 접속하기

위에서 사용한 kubectl attach 를 사용하면 실행중인 컨테이너에 대화형 세션을 생성할 수 있다. 이 커맨드는 팟이 활성화 되어있는 상태에서만 동작하고, 따라서 비정상 종료된 팟에 대해서는 접속이 불가하다.

### 컨테이너에서 두번째 프로세스 실행하기

kubectl exec 커맨드를 사용해 팟에 추가적인 커맨드를 실행할 수 있다.
```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl exec flask-665cf87fc4-pbmzg -it -- /bin/sh
/ # ls
bin      etc      lib      mnt      proc     root     sbin     sys      usr
dev      home     media    opt      python3  run      srv      tmp      var
```


## 쿠버네티스 콘셉트 - 라벨

쿠버네티스는 관리하는 객체를 연결하고 참조하기 위해 유연한 메커니즘을 가지고 있다. 서로 연결될 수 있는 리소스에 대해 계층 구조를 사용하지 않고 리소스에 대한 키-값 쌍으로 구성된 라벨이라는 개념을 사용한다. 또한 라벨에 대해서 쿼리하고 찾을 수 있도록 하는 셀렉터 라는 매칭 메커니즘이 존재한다.

라벨을 정의하기 위한 형식은 엄격하게 정의돼 있으며, 자원을 그룹화하기 위해 사용된다. 라벨은 어떤 고유한 리소스를 식별하기 위한 것이 아니라 팟, 레플리카셋, 디플로이먼트와 같이 쿠버네티스 리소스 집합에 대해 서로 연관된 정보를 설명하는데 사용된다.

라벨에서 키의 사이즈는 제한적이며 필요시 prefix 적용이 가능하다. 이때 라벨 키는 / 문자에 의해 prefix와 키 이름으로 구분된다. 라벨 키에 프리픽스가 적용된 경우 DNS 도메인이여야 한다. 쿠버네티스 내부 컴포넌트와 플러그인은 prefix를 사용해 라벨을 그룹화하거나 분리하며 kubernetes.io prefix는 쿠버네티스 내부 라벨에 사용하기 위해 예약되어 있다. prefix가 정의되지 않은 경우 완전히 사용자 통제하에 있기 때문에 사용자가 직접 prefix가 붙지 않은 라벨의 일관성을 위한 규칙을 관리해야 한다.

prefix는 253자 이하여야 하며 프리픽스를 제외한 나머지 키 이름은 최대 63자까지 가능하다. 라벨 키는 숫자, 알파벳, -, _ 문자만 가능하다.

라벨은 리소스에 의미 있는 정보를 나타내기 위해 사용되며 하나의 리소스가 여러개의 라벨을 갖는것 또한 가능하다.

라벨은 일반적으로 다음과 같은 정보를 나타낼 때 많이 사용된다.
- 환경
- 버전
- 어플리케이션 이름
- 서비스 계층

### 라벨의 구성
시스템을 구성하는 리소스가 한개 이상이라면 그룹화를 통해 시스템에 대한 이해를 도울 뿐 아니라 리소스를 개별 이름이나 ID가 아닌 역할에 따라 구분하는것이 가능하다. 

라벨의 이름만으로는 라벨의 역할에 대해 정확히 파악하기 힘들 수 있으니, 라벨의 의미와 의도를 배포 디렉토리의 README.md 와 같은 곳에 문서화해 정리를 하는것이 좋다.

## 쿠버네티스 콘셉트 - 셀렉터

kubectl -l 커맨드를 통해 검색한 정보를 필터링할 수 있는 셀렉터 기능을 제공한다.

### 동등 기반 셀렉터
- =
- !=

### 설정 기반 셀렉터
- in
- notin
- exist

### 라벨 확인

```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get pods -l app=flask
NAME                     READY   STATUS    RESTARTS   AGE
flask-665cf87fc4-pbmzg   1/1     Running   2          5d16h
```
```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl describe deployment flask
Name:                   flask
Namespace:              default
CreationTimestamp:      Sun, 21 Jun 2020 22:07:13 +0900
Labels:                 app=flask
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=flask
Replicas:               1 desired | 1 updated | 1 total | 1 
...
```
```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get deployments -l app=flask
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
flask   1/1     1            1           5d17h
```

또한 존재하는 리소스에 대해 kubectl label 커맨드를 사용해 대화형 방식으로 리소스에 라벨을 적용할 수 있다.

```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl label pods flask-665cf87fc4-pbmzg enable=true
pod/flask-665cf87fc4-pbmzg labeled
```
```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get pods -l enable=true
NAME                     READY   STATUS    RESTARTS   AGE
flask-665cf87fc4-pbmzg   1/1     Running   2          5d17h

```
### kubectl 커맨드를 사용해 라벨과 함께 리소스 정보 목록화하기

-L 옵션을 통해 리소스 정보를 함께 조회할 수 있다.
```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get pods -L run,pod-template-hash
NAME                     READY   STATUS    RESTARTS   AGE     RUN     POD-TEMPLATE-HASH
flask                    1/1     Running   2          5d17h   flask   
flask-665cf87fc4-pbmzg   1/1     Running   2          5d17h           665cf87fc4
```

## 쿠버네티스 리소스 - 서비스

서비스는 실행중인 특정 인스턴스에 관계없이 팟을 통해 추상화 계층을 제공하는 쿠버네티스 리소스다. 프론트엔드 웹 어플리케이션과 데이터베이스와 같이 서로 다른 계층(tier) 에 있는 컨테이너에 추상화 계층을 제공함으로써 쿠버네티스는 각 계층을 독립적으로 스케일하거나 업데이트를 수행할 수 있다. 또한 서비스 리소스는 어떠한 데이터를 전송할지 정의하는 정책을 포함할 수 있기 때문에 쿠버네티스에서 소프트웨어 로드 밸런서로 사용될 수 있다.

서비스는 팟 간에 상호 노출을 하거나 컨테이너를 쿠버네티스 클러스터 외부로 노출시킬 때 사용하는 핵심 추상화 요소다. 서비스는 쿠버네티스에서 팟 집함 간의 조정뿐만 아니라 팟 내외부에서 트래픽을 관리하는 핵심 요소다.

쿠버네티스 서비스 리소스의 진보적인 사용 방법은 전적으로 클러스터 외부에서 접속하는 리소스들을 위한 서비스를 정의해 사용하는 것이다. 이와 같은 방법은 엔드포인트가 쿠버네티스 내부인 경우이나 외부인 경우 관계없이 일관된 방법을 제공한다.

쿠버네티스에는 이미 실행중인 리소스를 기반으로 서비스를 생성하는 expose커맨드를 내장하고 있다.

### 서비스 리소스 정의

서비스의 명세는 YAML 이나 JSON 형식으로 간단하게 정의할 수 있다. 아래는 서비스 리소스를 정의한 예제이다
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

### 엔드포인트

서비스를 정의할 때 셀렉터는 필수 사항은 아니다. 셀렉터가 없는 서비스는 쿠버네티스 클러스터 외부에 있는 리소스를 위한 서비스를 의미한다. 쿠버네티스 클러스터 외부에 있는 리소스의 서비스를 생성하기 위해서는 셀렉터가 없는 서비스를 생성 후 엔드포인트라고 하는 새로운 유형의 리소스를 생성해야 한다. 엔드포인트는 원격 서비스의 네트워크상 위치를 의미한다.

아래는 엔드포인트 서비스 설정 예제이다.
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```
* 엔드포인트 오브젝트의 이름은 유효한 DNS 서브도메인 이름이어야 한다.

### 서비스 타입 - ExternalName

엔드포인트와 유사한 역할을 수행하지만 단순히 DNS 참조만을 제공하는 서비스가 ExternalName 서비스이다. 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

### 헤드리스 서비스

IP 주소를 할당하지 않거나 트래픽을 포워딩하지 않는 서비스 그룹

## 쿠버네티스 클러스터 외부로 서비스 노출하기

### 서비스 타입 - LoadBalancer

LoadBalancer는 모든 쿠버네티스 클러스터에서 지원하지 않는다. 클라우드 공급업체에서 제공하는 클라우드 환경에서 가장 많이 사용되며, 클라우드 공급업체의 인프라에서 제공하는 외부 로드 밸런싱 설정이필요하다.

### 서비스 타입 - NodePort

개발 시스템의 가상 머신에 미니큐브를 이용해 쿠버네티스 클러스터를 구축했을 경우 외부로 노출시키는 보편적인 방법이다. Nodeport는 로컬 네트워크에 액세스할 수 있도록 쿠버네티스 클러스터가 실행되는 호스트를 기반으로 동작하며 모든 쿠버네티스 클러스터 노드에서 높은 번호의 포트를 통해 서비스를 제공한다.

```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl expose deploy flask --port 5000 --type=NodePort
service/flask exposed
```

```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get service flask -o yaml | grep nodePort
  - nodePort: 30165
```

### 미니큐브 서비스

미니큐브는 서비스를 쉽게 생성하고 액세스할 수 있도록 하는 서비스 커맨드를 제공한다.
```
Shabreui-MacBook-Pro:kfd-flask shabre$ minikube service flask --url
http://192.168.64.2:30165
```

아래와 같이 커맨드를 실행할 경우 기본값으로 브라우저 창을 실행하는 유용한 옵션을 제공한다.

```minikube service flask
Shabreui-MacBook-Pro:kfd-flask shabre$ minikube service flask
|-----------|-------|-------------|---------------------------|
| NAMESPACE | NAME  | TARGET PORT |            URL            |
|-----------|-------|-------------|---------------------------|
| default   | flask |        5000 | http://192.168.64.2:30165 |
|-----------|-------|-------------|---------------------------|
🎉  Opening service default/flask in default browser...
```

## 실습

```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl create deployment redis --image=docker.io/redis:alpine
deployment.apps/redis created
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
flask   1/1     1            1           5d19h
redis   1/1     1            1           6s
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl expose deploy redis --port=6379 --type=NodePort
service/redis exposed
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get services
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
flask        NodePort    10.111.208.100   <none>        5000:30165/TCP   55m
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          7d2h
redis        NodePort    10.97.116.170    <none>        6379:31838/TCP   5s
```
```
Shabreui-MacBook-Pro:kfd-flask shabre$ minikube ip
192.168.64.2

Shabreui-MacBook-Pro:kfd-flask shabre$ redis-cli -h 192.168.64.2 -p 31838
192.168.64.2:31838> ping
PONG
```

## 디플로이먼트와 롤 아웃

디플로이먼트에서 이미지를 변경할 경우 롤 아웃이 발생한다. 디플로이먼트 롤 아웃은 완료하는데 시간이 소요되는 비동기 프로세스이며 디플로이먼트 내에서 정의된 값으로 제어된다. 리소스 파일의 spec -> strategy 하위 항목을 보면 업데이트 처리 방법에 관한 기본 사양을 확인할 수 있다.

```
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: flask
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```
롤링 업데이트는 maxSurge와 maxUnavailable 두가지 값에 의해 제어된다. 이 값은 업데이트가 롤 아웃되는 동안 서비스를 처리하기 위해 최소한의 팟의 수를 정의한다.

```
shabre@Shabreui-MacBook-Pro kfd-flask % kubectl replace -f flask_deployment.yaml

shabre@Shabreui-MacBook-Pro kfd-flask % kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
flask                    1/1     Running             3          7d1h
flask-665cf87fc4-pbmzg   1/1     Running             3          7d
flask-944cbc755-dfspq    0/1     ContainerCreating   0          32s
redis                    1/1     Running             1          30h
redis-64df8456c8-8tgwc   1/1     Running             1          29h

//롤아웃 완료 후

shabre@Shabreui-MacBook-Pro kfd-flask % kubectl rollout status deployment/flask
deployment "flask" successfully rolled out

shabre@Shabreui-MacBook-Pro kfd-flask % kubectl rollout history deployment/flask
deployment.apps/flask 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

shabre@Shabreui-MacBook-Pro kfd-flask % kubectl annotate deployment flask kubernetes.io/change-cause='deploying image 0.1.1'
deployment.apps/flask annotated
shabre@Shabreui-MacBook-Pro kfd-flask % kubectl rollout history deployment/flask                                            
deployment.apps/flask 
REVISION  CHANGE-CAUSE
1         <none>
2         deploying image 0.1.1
```

### 롤 아웃 실행 취소

리소스를 이전 버전으로 되돌릴 수 있는 기능이 존재한다.
```
shabre@Shabreui-MacBook-Pro kfd-flask % kubectl rollout undo deployment/flask
deployment.apps/flask rolled back

shabre@Shabreui-MacBook-Pro kfd-flask % kubectl rollout status deployment/flask -w
Waiting for deployment "flask" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "flask" rollout to finish: 1 old replicas are pending termination...
deployment "flask" successfully rolled out

shabre@Shabreui-MacBook-Pro kfd-flask % kubectl rollout history deployment/flask
deployment.apps/flask 
REVISION  CHANGE-CAUSE
2         deploying image 0.1.1
3         <none>
```

