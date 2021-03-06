# 02.도커 엔진

## 2.1 도커 이미지와 컨테이너
도커 엔진에서 사용하는 기본 단위는 **이미지**와 **컨테이너**이며, 이 두가지가 도커 엔진의 핵심이다.

### 2.1.1 도커 이미지
이미지는 컨테이너를 생성할 때 필요한 요소이며, 가상머신을 생성할 때 사용하는 iso파일과 비슷한 개념이다.
이미지는 여러개의 계층으로 된 바이너리 파일로 존재하고, 컨테이너를 생성하고 실행할 때 읽기전용으로 사용된다.
이미지는 도커 명령어로 내려받을 수 있으므로 별도로 설치할 필요는 없다.

도커에서 사용하는 이미지의 이름은 기본적으로 [저장소이름]/[이미지이름]:[태그] 의 형태로 구성된다.
- 저장소 이름 : 이미지가 저장된 장소. 
- 이미지 이름 : 해당 이미지가 어떤 역할을 하는지 나타낸다.
- 태그 : 이미지 버전 관리.

### 2.1.2 도커 컨테이너
도커 이미지로 컨테이너를 생성하면 해당 이미지의 목적에 맞는 파일이 들어있는 파일시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성되고, 이것이 바로 도커 컨테이너가 된다.

대부분 도커 컨테이너는 생성될 때 사용된 도커이미지의 종류에 따라 알맞은 설정과 파일을 갖고있기때문에
도커 이미지의 목적에 맞도록 사용되는 것이 일반적이다.
예를들어, 웹서버 도커 이미지로부터 여러개의 컨테이너를 설정하면 생성된 컨테이너 갯수만큼 웹서버가 생성되고 
이 컨테이너들은 외부에 웹서비스를 제공하는데 사용될 것이다.

컨테이너는 이미지를 읽기전용으로 사용하되 이미지에서 변경된 사항만 컨테이너 계층에 저장하므로 
컨테이너에서 무엇을 하든지 원래 이미지는 영향을 받지 않는다.
또한 생성된 각 컨테이너는 각기 독립된 파일시스템을 제공받으며 호스트와 분리돼 있으므로 특정 컨테이너에서 
어떤 애플리케이션을 설치하거나 삭제해도 다른 컨테이너와 호스트는 영향을 받지 않는다. 

## 2.2 도커 컨테이너 다루기 
도커엔진을 설치하고 컨테이너와 이미지의 기본적인 개념을 이해했다면, 도커엔진을 사용할 준비가 다 됐다.

### 2.2.1 컨테이너 생성
```
docker run -i -t ubuntu:14.04
```
docker run : 컨테이너를 생성하고 실행하는 역할을 한다.  
ubuntu:14.04는 컨테이너를 생성하기 위한 이미지 이름.  
-i옵션은 컨테이너와 상호입출력을 가능하게 하고,  
-t옵션은 tty를 활성화해서 배시셸을 사용하도록 컨테이너를 설정했다.  
ubuntu:14.04 이미지가 로컬 도커 엔진에 존재하지 않기때문에 도커 중앙 이미지 저장소인 도커 허브에서 자동으로 이미지를 내려받는다.   
단 한줄의 docker 명령어로 컨테이너 생성 및 실행과 동시에 컨테이너 내부로 들어왔다.  
컨테이너서 기본 사용자는 root이고 호스트이름은 무작위 16진수해쉬값입니다.  

ls로 파일시스템을 확인해보면 아무것도 설치되지 않은 상태를 확인할 수 있음.  
컨테이너 내부에서 호스트의 도커환경으로 돌아가는 방법은 exit 입력 or Ctrl+D 입력.  
단, 컨테이너 내부에서 빠져나오면서 동시에 컨테이너를 정지시킨다.  

컨테이너를 정지하지 않고 빠져나오는 방법 : Ctrl+P,Q (단순히 컨테이너 셸에서만 빠져나옴.). 

도커 이지미 목록 확인: docker images  

도커 공식 이미지저장소에서 이미지 내려받기 : docker pull centos:7. 

컨테이너를 생성할때는 create 명령어를 사용할 수도 있다.  
```
docker create -i -t --name mycentos centos:7
```
컨테이너 시작 : docker start mycentos  
컨테이너 내부로 들어가기 : docker attach mycentos   

### 2.2.2 컨테이너 목록 확인 
docker ps : 정지되지 않은 컨테이너만 출력함. 전체 출력시 -a 옵션.
- CONTAINER_ID
- IMAGE : 컨테이너 생성 시 사용된 이미지. 
- COMMAND : 컨테이너가 시작될 때 실행될 명령어. 
- CREATED
- STATUS 
- PORTS 
- NAMES

### 2.2.3 컨테이너 삭제
docker rm mycentos
docker stop mycentos  // 컨테이너 정지 한 뒤 삭제 가능.
docker containers prune // 모든 컨테이너 삭제.

### 2.2.4 컨테이너를 외부에 노출
컨테이너는 가상머신과 마찬가지로 가상IP를 할당받는다.
172.17.0.x 의 IP를 순차적으로 할당한다.

```
docker run -i -t --name network_test ubuntu:14.04
ifconfig
```
아무런 설정을 하지 않았다면, 이 컨테이너는 외부에서 접근할 수 없으며 도커가 설치된 호스트에서만 접근할 수 있다.
외부에 컨테이너의 애플리케이션을 노출하기 위해서는 eth0의 ip와 포트를 호스트의 ip와 포트에 바인딩 해야 한다.
컨테이너에서 호스트로 빠져나온 후 새로운 컨테이너에 아파치 웹서버를 설치해 외부에 노출시켜 보자.
```
docker run -i -t --name mywebserver -p 80:80 ubuntu:14.04
```
-p 옵션 : 컨테이너의 포트를 호스트의 포트와 바인딩해 연결할 수 있게 설정한다.
ex) 호스트의 7777포트를 컨테이너의 80포트와 연결하려면 7777:80 같이 입력.
여러 포트를 외부에 개방하려면 -p옵션을 여러번 쓴다.
```
docker run -i -t -p 3306:3306 -p [my ip]]:7777:80 ubuntu:14.04
```

컨테이너를 생성해 내부로 들어온 뒤 아파치 웹서버를 설치한다.
```
apt-get update
apt-get install apache2 -y
service apache2 start
```
브라우저에서 my ip:7777 로 아파치 웹서버 뜨는것을 확인!
호스트IP와 포트를 컨테이너의 IP와 포트로 연결한다는 개념은 매우 중요하다.
아파치 웹서버는 172대역을 가진 컨테이너의 NAT IP와 80포트로 서비스하므로 여기에 접근하려면
172.17.0.x:80의 주소로 접근해야 한다. 그러나 도커의 포트포워딩 옵션인 -p를 사용해서
호스트와 컨테이너를 연결했으므로 호스트의 ip와 포트를 통해 172.17.0.x:80으로 접근할 수 있다. 

### 2.2.5 컨테이너 어플리케이션 구축
대부분의 서비스는 단일 프로그램으로 동작하지 않는다.
여러 에이전트나 데이터베이스 등 연결되어 완전한 서비스로 동작하는 것이 일반적이다.
이런 서비스를 컨테이너 화 할 때 여러개의 어플리케이션을 하나의 컨테이너에 설치할 수도 있다.
그러나 컨테이너에 어플리케이션을 하나만 동작시키면 컨테이너간 독립성을 보장함과 동시에 
어플리케이션의 버전관리, 소스코드 모듈화 등이 더욱 쉬워진다.
한 컨테이너 안에 프로세스 하나만 실행하는 것이 도커의 철학이다.

- mysql 이미지를 사용해 데이터 베이스 컨테이너 생성 
```
docker run -d --name wordpressdb -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=wordpress mysql:5.7
```
- 워드프레스 이미지를 이용해 워드프레스 웹서버 컨테이너 생성 
```
docker run -d -e WORDPRESS_DB_HOST=mysql -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=password --name wordpress --link wordpressdb:mysql -p 80 wordpress
```

워드프레스 웹서버 컨테이너의 -p옵션에서 80을 입력햇으므로 호스트의 포트중 하나와 컨테이너 80포트가 연결된다.
docker ps 명령어로 호스트의 어느포트와 연결됬는지 확인이 가능하다. 
```
docker port wordpress
```
--link 옵션은 현재 deprecated 된 옵션이며 추후 삭제될 수 있어서 도커브리지 네트워크를 사용하면 동일한 기능을 사용할 수 있다. 

### 2.2.6 도커 볼륨
도커 이미지로 컨테이너를 생성하면 이미지는 읽기전용이 되며 컨테이너의 변경사항만 별도로 저장해서 각 컨테이너의 정보를 보존한다. 
이미 생성된 이미지는 어떠한 경우로도 변경되지 않으며, 컨테이너 계층에 원래 이미지에서 변경된 파일시스템 등을 저장한다.
이미지에 mysql 을 실행하는데 필요한 애플리케이션 파일이 들어있다면 컨테이너 계층에는 워드프레스에서 쓴 로그인 정보나 게시글 등과 같은 데이터베이스를 운영하면서 쌓이는 데이터가 저장된다.

그러나 mysql 컨테이너를 삭제하면, 컨테이너 계층에 저장돼 있던 데이터베이스 정보도 삭제된다.
도커의 컨테이너는 생성과 삭제가 매우 쉬우므로 실수로 컨테이너를 삭제하면 데이터를 복구할 수 없게 된다.
이를 방지하기 위해, 컨테이너의 데이터를 영속적 데이터로 활용할 수 있는 방법이 있는데 그중 가장 활용하기 쉬운 방법이 도커**볼륨**을 활용하는 것이다.

볼륨을 활용하는 방법에는 호스트와 볼륨을 공유하기, 볼륨 컨테이너를 활용하기, 도커가 관리하는 볼륨을 생성하기가 있다.

### 2.2.6.1 호스트 볼륨 공유 
### 2.2.6.2 볼륨 컨테이너
### 2.2.6.3 도커 볼륨

### 2.2.7 도커 네트워크
#### 2.2.7.1 도커 네트워크 구조
#### 2.2.7.2 도커 네트워크 기능

### 2.2.8 컨테이너 로깅
### 2.2.8.1 json-file 로그 사용하기
### 2.2.8.2 syslog 로그 
### 2.2.8.3 fluentd 로깅
### 2.2.8.4 아마존 클라우드워치 로그

### 2.2.9 컨테이너 자원 할당 제한

## 2.3 도커 이미지
### 2.3.1 도커 이미지 생성
### 2.3.2 이미지 구조 이해
### 2.3.3 이미지 추출
### 2.3.4 이미지 배포
### 2.3.4.1 도커 허브 저장소
### 2.3.4.2 도커 사설 레지스트리 

## 2.4 Dockerfile
### 2.4.1 이미지를 생성하는 방법
### 2.4.2 Dockerfile 작성
컨테이너에서 수행해야 할 작업을 명시한다.
### 2.4.3 Dockerfile 빌드
### 2.4.4 Dockerfile 명령어 
### 2.4.5 Dockerfile로 빌드할 때 주의할 점

## 2.5 도커 데몬
### 2.5.1 도커의 구조
### 2.5.2 도커 데몬 실행
### 2.5.3 도커 데몬 설정
### 2.5.4 도커 데몬 모니터링
### 2.5.5 Remote API 라이브러리를 이용한 도커 사용


