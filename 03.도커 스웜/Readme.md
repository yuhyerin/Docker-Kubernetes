# 03.도커 스웜

## 3.1 도커 스웜을 사용하는 이유
지금까지 알아본 도커 사용법은 대부분 하나의 호스트를 기준으로 한다.
docker ps 명령어는 하나의 도커엔진에 존재하는 컨테이너의 목록을 출력하며
create, run 명령어 또한 하나의 도커엔진에 컨테이너를 생성한다.
그러나 실제로 도커를 운영환경에 적용한다면 조금 이야기가 달라진다.
하나의 호스트 머신에서 도커엔진을 구동하다가 CPU, 메모리, 디스크용량 같은 자원이 부족하면 어떻게 해결할까?

가장 간단한 해답은 매우 성능이 좋은 서버를 새로 사면 된다.
하지만 자원의 확장성 측면이나 비용 측면에선 좋은 해답은 아니다.
자원이 부족할때마다 더 성능이 좋은 서버를 살수는 없을 뿐더러 서버를 사고 유지하는 비용도 무시할 수 없다.
이를 해결하기 위해 여러 방법이 제안됐지만, 가장 많이 사용하는 방법은 여러대의 서버를 클러스터로 만들어 
자원을 병렬로 확장하는 것이다. 

예를들어 8기가 메모리가 탑재된 서버3대에 도커엔진을 설치해 실제 운영환경에 사용한다고 가정해보자.
이 3대의 서버에 컨테이너가 너무 많이 생성돼 있어 더는 컨테이너를 사용할 수 없다고 판단되면,
8기가 메모리가 탑재된 서버를 추가해 자원을 늘린다.
이렇게 추가된 서버 자원만큼 클러스터 내의 가용자원은 늘어나므로 총 사용가능한 자원이 32기가가 된다.
따라서 필요한만큼 서버를 병렬로 확장해나가면 적당한 성능의 서버를 여러대를 하나의 자원풀로 만들어 사용할 수 있고
성능이 좋은 값비싼 서버를 사지 않아도 된다.

그러나 여러대 서버를 하나의 자원풀로 만드는 것이 쉬운 작업이 아니다.
새로운 서버나 컨테이너가 추가됐을 때, 이를 발견하는 작업부터 어떤 서버에 컨테이너를 할당할 것인가에 대한
스케줄러와 로드밸런서 문제, 클러스터 내 서버가다운됐을 경우 고가용성을 어떻게 보장할지 등이 문제로 남아있다.
이 문제를 해결하는 여러 솔루션을 오픈소스로 활용할 수 있는데, 이 가운데 대표적인것이 도커스웜과 스웜모드다.

## 3.2 스웜클래식과 도커 스웜모드
스웜 클래식과 스웜 모드는 여러대의 도커 서버를 하나의 클러스터로 만들어 컨테이너를 생성하는 여러 기능을 제공한다.
다양한 전략을 세워 컨테이너를 특정 도커서버에 할당할 수 있고, 유동적으로 서버를 확장할 수 있다.
그뿐만 아니라 스웜클러스터에 등록된 서버의 컨테이너를 쉽게 관리할 수 있다.
따라서 PaaS 같은 용도로 도커 서버 클러스터링을 고려하고 있다면, 가장 먼저 스웜을 사용해보는 것을 권장한다.
비록 도커 스웜모드가 실제 운영환경에서 많이 쓰이는것은 아니지만 서버 클러스터에서 컨테이너를 어떻게 다루는지에 대한 기초적인 지식을 쌓기에는 적합하기 때문이다.
서버 클러스터에서의 컨테이너 관리에 익숙하지 않다면 뒤에서 설명할 쿠버네티스를 읽기전에 스웜모드를 한번쯤 따라해보는 것도 나쁘지 않다.

도커 스웜에는 2가지 종류가 있다.
첫번째는 도커 버전 1.6이후부터 사용할 수 있는 컨테이너로서의 스웜, 
두번째는 도커버전 1.12 이후부터 사용할수 있는 도커 스웜모드다. 
첫번째를 스웜 클래식, 두번째를 스웜모드로 부르도록 하겠다.

스웜클래식과 스웜모드의 가장 큰 차이점은 목적에 있다.
스웜클래식은 여러대의 도커서버를 하나의 지점에서 사용하도록 단일 접근점을 제공한다.
스웜모드는 마이크로서비스 아키텍처의 컨테이너를 다루기 위한 클러스터링 기능에 초점을 맞추고 있다.

스웜클래식은 docker run , docker ps 등 일반적 도커 명령어와 도커API로 클러스터의 서버를 제어하고 관리할 수 있다.
스웜모드는 같은 컨테이너를 동시에 여러개 생성해 필요에 따라 유동적으로 컨테이너 수를 조절할 수 있으며, 컨테이너로의 연결을 분산하는 로드밸런싱 기능을 자체적으로 지원한다.
스웜모드가 서비스 확장성과 안정성 등 여러 측면에서 스웜 클래식보다 뛰어나기 때문에 일반적으로는 스웜모드를 더 많이 사용한다.

스웜클래식과 스웜모드의 다른 차이점은 분산 코디네이터, 에이전트와 같은 클러스터 툴이 별도로 구동되느냐 입니다.
여러개의 도커서버를 하나의 클러스터로 구성하려면 각종 정보를 저장하고 동기화하는 분산코디네이터, 클러스터 내 서버를 관리하고 제어하는 매니저, 각 서버를 제어하는 에이전트가 반드시 있어야 한다.
스웜클래식은 분산코디네이터, 에이전트 등이 별도로 실행돼야 하지만, 스웜모드는 클러스터링을 위한 모든 도구가 도커엔진 자체에 내장돼 있기 때문에 더욱 쉽게 서버 클러스터를 구축할 수 있다.

- 분산 코디네이터 : 클러스터에 새로 영입할 새로운 서버의 발견, 클러스터 각종 설정 저장, 데이터 동기화 등에 주로 이용됨.
ex) etcd, zookeeper, consul
스웜클래식은 대부분 분산코디네이터를 사용할 수 있으며, 스웜모드는 별도로 분산코디네이터를 구축하지 않아도 된다.
- 매니저 노드 ( 에이전트 컨테이너 )
- 워커노드 ( 에이전트 컨테이너 )

스웜모드는 MSA 구조의 애플리케이션을 컨테이너로 구축할 수 있도록 도와줄 뿐만 아니라, 서비스 장애에 대비한 고가용성과 부하분산을 위한 로드밸런싱 기능을 제공해주므로 가능하면 스웜모드를 사용하는 것을 추천한다!

스웜모드는 적어도 3대이상의 도커서버를 사용해야만 기능을 제대로 테스트해볼 수 있다.
스웜클래식은 도커공식문서에서도 레거시로 언급하고있으며, 유지보수가 활발하지 않아서 다루지 않겠다.

후반부에서 다룰 쿠버네티스는 스웜모드와 비슷하지만, 더욱 복잡하고 많은 기능을 제공하는 솔루션이다.
따라서 마이크로서비스, 컨테이너 클라우드 등에대한 기본개념이 없으면 쿠버네티스를 이해하기엔 어렵다.
스웜모드는 클라우드에서 마이크로서비스 아키텍처의 컨테이너가 어떻게 배포되고 사용될 수 있는지를 학습하기에 나쁘지 않은 솔루션이므로 스웜모드를 이해한다면 쿠버네티스를 이해하는 것이 좀 더 수월할 것이다~! 

## 3.3 스웜모드
스웜모드는 별도의 설치과정이 필요치 않으며, 도커 엔진 자체에 내장되어 있다.
docker info 명령어를 통해 스웜모드 클러스터 정보를 확인할 수 있다.
```
docker info
```
지금까지 도커엔진을 사용해온 방법들은 전부 단일도커서버에서 사용된 것이므로, 현재 스웜모드는 inactive 상태로 설정돼 있다.

### 3.3.1 도커 스웜모드의 구조
스웜모드는 **매니저 노드**와 **워커 노드**로 구성돼 있다.
