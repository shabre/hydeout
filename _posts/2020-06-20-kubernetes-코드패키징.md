---
layout: post
title: kubernetes 코드 패키징
categories: kubernetes
excerpt_separator: "<!--more-->"
---

## 도커파일 빌드 주요 커맨드

### FROM
컨테이너 이미지를 빌드할 때 사용하는 베이스 이미지를 의미함.

### RUN
컨테이너 이미지 빌드 과정에서 실행하는 커맨드. 베이스 이미지에 디펜던시나 라이브러리를 추가하는데 많이 사용됨.

### ENV
컨테이너 내부에서 소프트웨어를 실행하기 전에 환경변수를 설정하는 역할을 함. ENV를 사용하게 되면 컨테이너 이미지를 생성하는 과정에서 환경변수가 설정되어 예기치 않은 결과가 발생할 수 있음. ENV 대신 RUN 커맨드를 사용하여 환경변수를 설정하는것이 권장됨.

### COPY
컨테이너 이미지에 호스트에 있는 로컬파일을 복사하는데 사용됨. COPY 커맨드는 컨테이너 이미지에 코드를 복사하는 가장 효과적인 방법.

### WORKDIR
컨테이너 내부에 로컬 디렉터리를 생성하고 해당 디렉터리를 WORKDIR 이후에 나오는 모든 커맨드 실행의 작업 디렉터리로 설정한다.

### LABEL
docker inspect 커맨드를 통해 볼 수 있는 값을 설정

### CMD, ENTRYPOINT
컨테이너 이미지를 실행할 때 함께 실행해야 할 커맨드와 스크립트 등을 명시한다.

## 컨테이너 만들기
도커 컨테이너는 Dockerfile 이라는 이름의 파일을 생성하고, 커맨드를 작성하면 된다.

```
From alpine #alpine linux 이미지
CMD ["/bin/sh", "-c", "echo 'hello world'"] #echo hello world 실행
```

도커파일을 작성 후에 해당 디렉토리에서 아래와 같이 빌드 명령을 수행한다.
```
docker build .
````

빌드가 성공적으로 완료되면 아래와같은 메세지가 출력이 된다.
```
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM alpine
 ---> a24bb4013296
Step 2/2 : CMD ["/bin/sh", "-c", "echo 'hello world'"]
 ---> Running in 2548dabf8d0e
Removing intermediate container 2548dabf8d0e
 ---> 0bb8bb5b624f
Successfully built 0bb8bb5b624f
```

마지막줄을 보면 0bb8bb5b624f 가 출력되었는데, 이것이 빌드 결과로 생성된 이미지의 ID 값이다. 빌드된 이미지는 아래의 명령어로 실행 가능하다.

```
docker run 0bb8bb5b624f
```
실행을 하면 hello world 가 출력이 되는것을 볼 수 있다.

생성되었던 이미지를 삭제하려면 아래와 같이 명령어를 입력하면 된다.
```
docker rmi -f 0bb8bb5b624f
```

## python-flask 컨테이너 이미지 예제 빌드

쿠버네티스를 실제로 어떻게 사용하는지 알아보기위해 예제로 만들어져있는 python-flask 이미지를 통해 실습을 진행한다.
아래 깃 리모트 레포에서 프로젝트를 클론한다.
```
git clone https://github.com/kubernetes-for-developers/kfd-flask
cd kfd-flask
git checkout tags/first_container
```

폴더 안에 보면 미리 만들어져있는 도커파일이 있다. 정의되어있는 도커파일을 살펴보면 마지막줄의 CMD 를 실행하기 전에 여러가지의 환경세팅을 하는것을 볼 수 있다. 도커파일을 만드는 목적은 아래와 같다.

- 기본이 되는 OS의 보안 패치 다운로드와 설치
- 코드 실행에 필요한 프로그래밍 언어와 실행 환경 설치
- 코드 실행에 필요한 디펜던시 설치
- 코드 컨테이너 내부로 복사
- 실행 대상 및 실행 방법 정의

실습 예제의 도커파일은 아래와 같다.
```
FROM alpine
# load any public updates from Alpine packages
RUN apk update
# upgrade any existing packages that have been updated
RUN apk upgrade
# add/install python3 and related libraries
RUN wget https://pkgs.alpinelinux.org/package/edge/main/x86/python3
RUN apk add python3
RUN apk add py3-pip
# make a directory for our application
RUN mkdir -p /opt/exampleapp
# move requirements file into the container
COPY . /opt/exampleapp/
# install the library dependencies for this application
RUN pip3 install -r /opt/exampleapp/requirements.txt
ENTRYPOINT ["python3"]
CMD ["/opt/exampleapp/exampleapp.py"]                                      
```

도커를 빌드하면 전과 같이 이미지 ID가 생성된다. 생성된 ID는 da95e7ec78c8 이다.
이렇게 생성된 이미지를 리모트 레포지토리에 push를 할 수 있다. 

리모트 레포지토리에 푸시를 하기 전에 리모트 레포지토리 계정을 하나 만들어야 한다. https://quay.io/ 로 접속하여 회원가입 후 flask 레포지토리를 하나 생성한다. 나의 경우 생성된 레포의 url은 다음과 같다.

https://quay.io/repository/shabreking92/flask

docker에서 지원하는 login 커맨드를 이용하여 quay.io에 접속하고, 푸시할 이미지의 태그와 버전을 명시한 다음 푸시를 진행해보자.
```
docker login quay.io
docker tag da95e7ec78c8 quay.io/shabreking92/flask:0.1.0 #:0.1.0 을 통해 버전 명시
docker push quay.io/shabreking/flask
```

푸시가 성공적으로 되었다면 내 레포지토리에 도커 이미지가 성공적으로 업로드되어있는것을 확인할 수 있다.

개발 과정에서 지속적으로 컨테이너 이미지에 코드를 올리고 배포하는 과정을 반복하기 때문에 컨테이너 이미지를 빌드하는 과정은 자동화 시켜놓는것이 좋다. 아래는 일반적으로 수행되는 컨테이너 이미지 빌드 프로세스다.

1. 소스코드 저장소에서 코드 다운로드
2. docker build 커맨드 수행
3. docker tag 커맨드 수행
4. docker push 커맨드 수행

## python-flask 컨테이너 이미지 예제 배포

kubectl create deployment 명령으로 이미지를 배포할 수 있다.
```
kubectl create deployment flask --image=quay.io/shabreking92/flask:latest
```

배포된 결과는 아래의 deployments, replicaset, pods 를 통해 알아볼 수 있다.
```
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
flask   0/1     1            0           15s
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
flask-665cf87fc4   1         1         0       61s
Shabreui-MacBook-Pro:kfd-flask shabre$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
flask-665cf87fc4-pbmzg   1/1     Running   0          73s
```

배포된 이미지를 expose 명령어를 통해 외부로 노출되도록 설정하고, minkube service 명령어를 통해 실제 실행시킬 수 있다..
```
kubectl expose deployment flask --type=LoadBalancer --port=5000
minikube service flask
```

```
|-----------|-------|-------------|---------------------------|
| NAMESPACE | NAME  | TARGET PORT |            URL            |
|-----------|-------|-------------|---------------------------|
| default   | flask |        5000 | http://192.168.64.2:31136 |
|-----------|-------|-------------|---------------------------|
🎉  Opening service default/flask in default browser...
```

## 포트 포워딩

포트포워딩을 할 경우 지정된 로컬 포트와 expose된 도커 포트로 포워딩 시켜줄 수 있다. 실행되는 동안은 포그라운드 프로세스로 진행되어 다른 명령어 입력은 불가하다. 명령어는 아래와 같다.
```
kubectl port-forward flask-665cf87fc4-pbmzg 5000:5000
```

실행 결과는 아래와 같다
```
Forwarding from 127.0.0.1:5000 -> 5000
Forwarding from [::1]:5000 -> 5000
```

## 프록시
쿠버네티스 클러스터 외부에서 팟에 접근하기 위해 사용 가능한 또 다른 커맨드이다. 이 커맨드는 팟에 대한 접근뿐만 아니라 쿠버네티스 API 에 대한 접근도 가능하도록 제공한다. 포트포워딩과 마찬가지로 실행중에는 다른 명령어 입력이 불가하다. 실행 커맨드는 아래와 같다.
```
kubectl proxy
```
