# Docker Compose 로 Jenkins 설치하기. 

Docker Compose 를 이용하여, 하나의 host 에 여러 Docker Container 를 띄우고, 컨테이너를 함께 동작하도록 할 수 있습니다. 

이번에는 mac 에 Docker 를 이용하여 Jenkins 를 설치해 보겠습니다. 

## Mac 에 Docker 설치하기. 

Docker 는 기본적으로 Linux 계열에서 동작합니다. 그러나 Mac 에서 Docker Desktop 을 설치하면 Docker 를 이용할 수 있습니다. 

설치는 https://docs.docker.com/docker-for-mac/install/ 에서 설치해 주면 됩니다. 

## Jenkins master 설치하기. 

Jenkins master 를 설치하기 위해서는 docker-compose.yml 을 이용해서 설치 할 수 있습니다. 

docker-compose를 이용하는 이유는 jenkins master - slave 구조로 이용 하기 위해서 입니다. 

일반적으로 jenkins master 에서는 특정 job 을 트리거링 하고, 이를 slave 가 받아서 job 을 수행하도록 구성해서 이용합니다. 

### docker-compose.yml 작성하기

master 을 실행하기 위해서 docker-compose.yml 을 작성합니다. 

```yaml
version: "3"
services: 
  master:
    container_name: master
    image: jenkins/jenkins:lts
    ports:
      - 8080:8080
    volumes: 
      - ./jenkins_home:/var/jenkins_home
```

- version: docker-compose 버젼은 3을 이용합니다. 
- services: 는 docker-compose 가 정의할 서비스이며, 이 서비스 하위에 서비스를 등록하여 어떠한 서비스가 수행될지를 지정하는 디렉티브 입니다. 
- master: 서비스 이름입니다. 
- container_name: 컨테이너 이름을 지정합니다. 이는 docker --name <이름> 에 해당하는 이름입니다. 
- image: 도커 이미지를 설정합니다. 우리는 [jenkins/jenkins:lts](https://hub.docker.com/r/jenkins/jenkins) 을 이용합니다. 
- ports: 포트 포워딩을 해줄 포트를 이용합니다. 
    - 포트 포워딩은 '-' 을 통해서 배열로 작성할 수 있습니다. 
    - 우리는 <호스트포트>:<docker container 포트> 로 나열합니다. 
    - 이렇게 되면 우리는 외부에서 호스트 포트를 이용하여 http://localhost:8080 을 통해서 접속할 수 있고, 호스트 포트로 접속하면 컨테이너 내부의 8080 포트로 접근됩니다. 
- volumes: 볼륨은 컨테이너 내부의 특정 디렉토리와 호스트 디렉토리를 연결하도록 해줍니다. 
    - <호스트 디렉토리경로>:<docker container 포트> 로 작성합니다. 
    - 이렇게 되면 docker 컨테이너가 올라오면 컨테이너내부의 /var/jenkins_home 에 작성되는 정보가, 우리가 사용하는 docker-compose를 실행할 디렉토리의 jenkins_home 과 연동됩니다. 

### 실행 확인하기. 

이제는 실제 docker-compose 를 실행해 보겠습니다. 

우선은 jenkins master 하나만 올립니다. 

```bash
$ cd <docker-compose.yml이 있는 디렉토리>
$ docker-compose -f docker-compose.yml up

Creating network "jenkins-install_default" with the default driver
Creating master ... done
Attaching to master
master    | Running from: /usr/share/jenkins/jenkins.war
master    | webroot: EnvVars.masterEnvVars.get("JENKINS_HOME")
master    | 2020-04-14 01:05:50.060+0000 [id=1]	INFO	org.eclipse.jetty.util.log.Log#initialized: Logging initialized @753ms to org.eclipse.jetty.util.log.JavaUtilLog

...
master    | *************************************************************
master    |
master    | Jenkins initial setup is required. An admin user has been created and a password generated.
master    | Please use the following password to proceed to installation:
master    |
master    | 73a7f4bb45a2457d8f576e453422350b
master    |
master    | This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
master    |
master    | *************************************************************
...
```

위에서 나오는 암호를 복사해 둡니다. 

jenkins 가 설치되면 localhost:8080 으로 접속하면 id/password 를 입력하라고 나타납니다. 

- id: admin
- password: 73a7f4bb45a2457d8f576e453422350b  <-- 위에서 복사한 비번입니다.

![docker-jenkins01](./imgs/docker-jenkins01.png) 

여기에 암호를 입력해 주고 continue 버튼을 클릭합니다.

### Jenkins Plugin 설치하기. 

이제 Jenkins Plugin 을 설치합니다. 

"Install suggested plugins" 을 클릭합니다. 

![docker-jenkins02](./imgs/docker-jenkins02.png) 

그럼 다음과 같이 설치가 진행됩니다. 

![docker-jenkins03](./imgs/docker-jenkins03.png) 

이 작업은 시간이 걸리니 커피한잔 하고 오면 좋습니다. 

이후 작업은 다음과 같이 진행하면 됩니다. 

![docker-jenkins04](./imgs/docker-jenkins04.png) 

![docker-jenkins05](./imgs/docker-jenkins05.png) 

![docker-jenkins06](./imgs/docker-jenkins06.png) 

![docker-jenkins07](./imgs/docker-jenkins07.png) 


### SSH 접속 키 생성

jenkins master 는 이벤트가 트리거링 되면 slave 로 잡을 수행하도록 전달합니다. 

이때 ssh 키가 있어야 master 가 slave 에 접속하여 통신할 수 있습니다. 

```bash
% docker container exec -it master ssh-keygen -t rsa -C ""

Generating public/private rsa key pair.
Enter file in which to save the key (/var/jenkins_home/.ssh/id_rsa):
Created directory '/var/jenkins_home/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/jenkins_home/.ssh/id_rsa.
Your public key has been saved in /var/jenkins_home/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:GtG/WuYP9C13EKBT420QQTUFG0WO+oox01ZkYgi/P9k
The key's randomart image is:
+---[RSA 2048]----+
|        .  .O+==+|
|       . o = = * |
|      . . = + O .|
|       . . + * . |
|      . S + . o  |
|       o . = * . |
|      .   O O E .|
|         = B = . |
|        . o.o    |
+----[SHA256]-----+
```

위와 같이 실행합니다. 

이렇게 실행하면 /var/jenkins_home/.ssh 디렉토리에 인증키와 퍼블릭키가 저장됩니다. 

```bash
% docker container exec -it master bash

jenkins@19f48a9c0bbb:/$ cd /var/jenkins_home/.ssh
jenkins@19f48a9c0bbb:~/.ssh$ ls
id_rsa	id_rsa.pub
```

위처럼 해당 파일이 있는 것을 확인했습니다. 

이제 PUBLIC 키를 복사해 둡니다. 

### 환경변수 설정하기. 

이제는 환경변수를 설정하겠습니다. 

docker-compose 은 실행시 .evn 파일에 존재하는 프로퍼티에 접근하여 환경변수를 확인할 수 있습니다. 

.env 파일

```text
JENKINS_SLAVE_SSH_PUBKEY=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAA ...
```

위와 같이 작성해 둡니다. 

## Slave 추가하기. 

이제 jenkins slave 를 docker-compose.yml 에 작성해 보겠습니다. 

ssh 로 접근하는 jenkins slave 이므로 다음 [docker 이미지](https://hub.docker.com/r/jenkins/ssh-slave)를 이용합니다. 

우리는 2개의 slave 를 만들어 줄 것입니다. 

docker-compose.yml 파일을 다음과 같이 작성 해줍니다. 

```yaml
version: "3"
services: 
  master:
    container_name: master
    image: jenkins/jenkins:lts
    ports:
      - 8080:8080
    volumes: 
      - ./jenkins_home:/var/jenkins_home
    # links 를 통해서 서비스끼리 연동할 수 있습니다. 
    links:
      - slave01
      - slave02
  # jenkins - slave 를 정의합니다. 
  # 환경변수를 아래와 같이 JENKINS_SLAVE_SSH_PUBKEY 로 접근 할 수 있습니다.
  slave01:  
    container_name: slave01
    image: jenkins/ssh-slave
    environment: 
      - JENKINS_SLAVE_SSH_PUBKEY= "${JENKINS_SLAVE_SSH_PUBKEY}"

  slave02:
    container_name: slave02
    image: jenkins/ssh-slave
    environment: 
      - JENKINS_SLAVE_SSH_PUBKEY= "${JENKINS_SLAVE_SSH_PUBKEY}"
```

### docker-compose 내리고 다시 실행하기. 

```bash
$ cd <docker-compose.yml이 있는 디렉토리>
$ docker-compose -f docker-compose.yml down
...

$ docker-compose -f docker-compose.yml up
...

```

### 컨테이너 확인하기. 

성상으로 컨테이너가 실행되었는지 확인해 봅니다. 

```bash
% docker-compose ps
 Name                Command               State                 Ports
------------------------------------------------------------------------------------
master    /sbin/tini -- /usr/local/b ...   Up      50000/tcp, 0.0.0.0:8080->8080/tcp
slave01   setup-sshd                       Up      22/tcp
slave02   setup-sshd                       Up      22/tcp
```

보시는 바와 같이 master 1개, slave 2개가 떴습니다. 

### jenkins 노드 등록하기. 

이제 slave01, slave02 를 마스터 노드와 연결하겠습니다. 

우선 키를 복사해야합니다. 

```bash
% docker container exec -it master bash
jenkins@1d9755ab7ca1:/$ cd /var/jenkins_home/.ssh
jenkins@1d9755ab7ca1:~/.ssh$ ls
id_rsa	id_rsa.pub
jenkins@1d9755ab7ca1:~/.ssh$ cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAwjv+W6ThWu5lnAut5mT9ixO8JBZNShfvHoDCa0VA17qtJjI1
ee93NPu3lcKos3UOp6E+UhBOqku/kIC5G6naBqSG3nSVJ3uMZoN+nFPVgGo1lVxk
...
```

위와 같이 -----BEGIN RSA PRIVATE KEY----- 전문을 복사해 둡니다. 

#### jenkins 에 접속하여 노드 등록하기. 

![docker-jenkins08](./imgs/docker-jenkins08.png) 

에서 Jenkins 관리를 선택합니다. 

![docker-jenkins09](./imgs/docker-jenkins09.png) 

"노드 관리" 를 찾아 선택합니다. 

![docker-jenkins10](./imgs/docker-jenkins10.png) 

에서 "신규 노드" 를 클릭하고, 위 내용과 같이 내용을 입력합니다. 

![docker-jenkins11](./imgs/docker-jenkins11.png) 

위 내용에서와 같이 

- Name: slave01
- Remote root directory: /home/jenkins
- Launch method: Launch agents via SSH
    - Host: slave01

그리고 Credentials 에서 Add 버튼을 클릭합니다. 

![docker-jenkins12](./imgs/docker-jenkins12.png)

위 창이 열리면 다음과 같이 선택합니다. 

- Domain: Global credentials (unrestricted)
- Kind: SSH Username with private key
    - Scope: Global (Jenkins, nodes, items, all child items, etc)
    - Username: jenkins
    - Private Key: Enter directly
        - Key: 위에서 복제한 RSA PRIVATE KEY 내용을 붙여넣어 줍니다. 

"Add" 버튼을 클릭합니다. 

![docker-jenkins13](./imgs/docker-jenkins13.png)

나머지 설정 부분을 지정합니다. 

- Credentials: jenkins 를 선택합니다. 
- Host Key Verification Strategy: Non verifying Verification Strategy

- Environment variables
    - 이름: JAVA_HOME
    - 값: /usr/local/openjdk-8

이후에 "Save" 버튼을 클릭합니다. 

![docker-jenkins14](./imgs/docker-jenkins14.png)

정상으로 확인되었습니다. 

이제 slave02 도 동일한 방식으로 연동해 주면 됩니다. 

## 결론

지금까지 docker-compose 를 통해서 jenkins 를 설치해 보았습니다. 

jenkins master/slave 를 서로 연동하기 위해서 docker-compose 를 이용하였고, 

links / service 이름을 통해서 상호 연결하였습니다. 

그리고 master 가 slave 에 자유롭게 접근하기 위해서 ssh 를 이용하여 연동하는 것도 해 보았습니다. 

