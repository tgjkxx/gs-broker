# gs-broker

### Overview

엣지 서비스 플랫폼인 'gedge-platform'에서 활용할 수 있는 엣지 응용 서비스들과  IoT 디바이스 간의 이종 프로토콜을 통합하여 중개할 수 있는 메시지 브로커이다. 

이 메시지 브로커는 6종의 통신 프로토콜 사용을 지원하며, 해당 프로토콜을 통해 IoT 디바이스 및 애플리케이션과 엣지 응용 서비스 사이의 실시간 통신이 가능하다. 지원하는 6종의 통신 프로토콜은 다음과 같다.

- SMQP
- REST
- MQTT
- STOMP
- WEB_STOMP
- WEB_MQTT


### Architecture

엣지 서비스 플랫폼에서 사용할 수 있는 메시지 브로커의 구조는 다음과 같다. 이는 MQTT 브로커의 개념을 확장시켜 이종 프로토콜 또한 지원할 수 있도록 구성되었다.

![message-broker-architecture](https://user-images.githubusercontent.com/70132781/100324978-4bd5ac00-300b-11eb-87bd-eb0b787d040c.png)

- Message Subscription : 메시지를 구독하는 주체로, 다양한 디바이스에서부터 전달되는 6종의 프로토콜 메시지를 구독하는 역할을 한다.
- Message Broker : 전달된 메시지를 gRPC 프로토콜로 변경해 주는 중간 역할을 한다.
- Message Publication : 메시지를 발급하는 주체로, 엣지 서비스에 메시지를 전달하는 역할을 한다.


# channel-creator
![image](https://github.com/jinsung-cho/KubeMQ-Web-Channel-Creator/assets/57334203/8936467c-8c64-4dcb-93e0-c8aa3180d719)
- 로컬 네트워크에서 KubeMQ-Community의 실행 및 MQ, Message 관리를 위한 Frontend와 Backend의 자동화된 실행을 제공한다.
- KubeMQ에 Channel 생성을 위한 다섯가지 메시지 프로토콜을 제공하고 메시지 전송으로 Channel을 생성할 수 있는 기능을 제공한다.
- 원격의 접근 가능한 네트워크에서 동작중인 KubeMQ-Community를 등록한 뒤 Channel 생성 및 메시지 전송할 수 있는 관련 기능을 제공한다.


## Requirements
- Docker Compose version 2.0.0 or higher

## 디렉토리 구조

프로젝트의 디렉토리 구조는 다음과 같다:

- `backend/`: go로 작성된 backend Rest API server 관련 코드가 포함된 디렉토리
- `frontend/`: react web 서비스를 위한 코드가 포함된 디렉토리
- `docker-compose.yaml`: backend, frontend, db, kubemq 컨테이너화 및 실행 자동화를 위한 docker-compose파일
- `start.sh`: env 파일 설정 및 docker compose 자동화를 위한 스크립트 파일

### backend

`backend` 디렉토리에는 다음과 같은 파일이 포함되어 있습니다.

- controller/ : Web request를 처리하기 위한 코드가 포함된 디렉트로
- model/ : DB 연결 및 쿼리를 위한 코드가 포함된 디렉토리
- router/ : Http request를 적절한 controller의 함수로 routing 하기 위한 코드가 포함된 디랙토리
- util/ : Error handling을 위한 코드가 포함된 디렉토리
- main.go : Golang 기반 backend server를 실행하기 위한 메인 코드
- Dockerfile : docker-compose 명령어 수행시 golang 기반 backend server를 docker image로 빌드하기 위한 Dockerfile
- .env-sample : DB 연동을 위한 여러 환경변수를 포함한 파일
  
### frontend

`frontend` 디렉토리에는 다음과 같은 파일이 포함되어 있다.

- public/
- src/
  - layouts/ : component들의 집합
  - App.js : 최상위 컴포넌트
  - axiosInstance.js : axios 설정을 관리하기 위한 js파일
  - index.js : 최상위 컴포넌트 App.js 를 랜더링 하기 위한 파일
  - routes.js : 페이지 routing을 위한 url 등 설정 값이 작성된 파일
- Dockerfile : docker-compose 명령어 수행시 React web server를 docker image로 빌드하기 위한 Dockerfile
- 이하 생략...
   
### docker-compose.yaml

`docker-compose.yaml` 파일에 대한 내용과 설명은 다음과 같다.
```
version: "1"
services:
  front: # react-web을 컨테이너로 실행하기 위한 설정내용
    build: "./frontend"
    container_name: react-web
    restart: always
    ports:
      - "3000:3000"
    depends_on:
      - backend
    links:
      - backend

  backend: # golang backend server를 컨테이너로 실행하기 위한 설정내용
    build: "./backend"
    container_name: go-server
    restart: always
    ports:
      - 8000:8000
    depends_on:
      - db
      - kubemq
    networks:
      - default

  db: # mysql db 컨테이너를 실행하기 위한 설정내용 (mysql version 5.7 or higher)
    image: mysql:5.7
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 1234
    ports:
      - "3306:3306"
    networks:
      - default
    volumes:
      - ./db_data:/var/lib/mysql

  kubemq: # kubemq-community 컨테이너를 실행하기 위한 설정 내용
    image: kubemq/kubemq-community:latest
    container_name: kubemq
    ports:
      - "8080:8080"
      - "50000:50000"
      - "9090:9090"
    restart: always
```

### start.sh
`start.sh` 파일의 내용과 설명은 다음과 같다.
```
#!/bin/sh

# Host IP를 .env에 추가하기 위한 내용
sed -i '/^HOST_IP/d' ./backend/.env
echo "\nHOST_IP=`hostname -I | awk '{print $1}'`" >> ./backend/.env
echo "REACT_APP_HOST_IP=`hostname -I | awk '{print $1}'`" > ./frontend/.env

# docker-compose.yaml 실행
docker-compose up --build 
```


## 실행방법
1. `$ git clone [ssh url] or [https url]`
2. `$ cd KubeMQ-Web-Channel-Creator/backend && mv .env-sample .env`
   - docker-compose.yaml 파일에 작성된 db관련 내용을 수정하지 않을 경우 (권장)
4. `$ cd ..`
5. `$ ./start.sh`
6. localhost:3000 접속
