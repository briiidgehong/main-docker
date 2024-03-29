# docker-k8s

## main concept
- 동일한 이미지에 기반한 다수의 컨테이너가 완전히 격리된 환경을 가지는 것

## docker volume concept
  ![image.jpg1](https://user-images.githubusercontent.com/73451727/188522927-03baca1a-93d3-45d4-a6b0-c3fd7dc10083.png) |![image.jpg2](https://user-images.githubusercontent.com/73451727/189511780-b505af93-d771-42c9-a7fb-3687d984ff83.png)
--- | --- | 
<img width="883" alt="스크린샷 2022-09-12 오후 4 22 05" src="https://user-images.githubusercontent.com/73451727/189595913-e731bd2f-55ef-4141-a121-431a090df19e.png">



- annonymous volume
  
  docker run -v app/feedback {container_path}
  
  컨테이너 삭제시 같이 삭제됨 - 하나의 컨테이너에 연결되어있음 - 컨테이너간 데이터 공유 불가능
  
  익명볼륨은 외부 경로보다 컨테이너 내부 경로의 우선 순위를 높이는데 사용할 수 있다.
  (컨테이너에 이미 존재하는 특정 데이터를 잠그는데 유용하다. 즉, 해당 볼륨의 데이터가 다른 볼륨이나 명령어에 의해 덮어쓰여지는것을 방지한다.)
 
- named volume
  
  docker run -v feedback:/app/feedback {volume_name:container_path}
  
  컨테이너 삭제시 삭제되지 않음
  
    - 하나의 컨테이너에 종속되지 않음 

    - 다수의 다양한 컨테이너에 동일하게 명명된 볼륨 하나를 마운트 할 수 있다.

    - 컨테이너간 데이터 공유 가능 
  
  편집이 불가하다. 볼륨이 실제 호스트 머신의 어디에 저장되어있는지 모르므로
  
- bind mount
  
  docker run -v ${pwd}:/app/feedback {호스트상의 절대경로:container_path}
  
  컨테이너 삭제시 삭제되지 않음
  
    - 하나의 컨테이너에 종속되지 않음 

    - 다수의 다양한 컨테이너에 동일하게 명명된 볼륨 하나를 마운트 할 수 있다.

    - 컨테이너간 데이터 공유 가능 
  
  호스트 머신상의 경로를 컨테이너와 맵핑시킨다.
  
  컨테이너에 라이브데이터를 제공한다. 이미지를 리빌드 할 필요가 없다.
  
  변경사항이 생기면, 소스코드를 스냅샷에서 복사하는것이 아니라 바인드마운트에서 복사함
 

```
docker build . -t docker-volume-test
docker run -d -p 3000:80 --rm docker0-volume-test
docker run -d -p 3000:80 -v app/feedback
docker run -d -p 3000:80 -v feedback:app/feedback
docker run -d -p 3000:80 -v ${pwd}:/app/feedback


# docker run 시점에서, bind mount를 시행하면 Dockerfile에서 
# COPY package.json .
# RUN npm install
# COPY . . 
# 했던 모든 파일들이 날아가고 로컬파일로 덮어씌워지게된다. (로컬 -> 컨테이너 방향)
# 특정 파일을 잠그려면, 더 구체적인 경로의 볼륨을 설정해주면 된다. (도커에서 판단할때, /app보다 /app/module 볼륨의 우선순위가 더 높다.)
# VOLUME ["app/node_modules"] or -v /app/node_modules

docker run -d -p 3000:80 --name feedback-app 
-v feedback:/app/feedback # named_volume
-v ${pwd}:/app:ro # bind mount # read only - 컨테이너가 마운트된 로컬 데이터를 변경하지 못하도록, 읽어갈수만 있도록 강제함
# -v /app/node_modules # file locking, 덮어쓰지 않도록 잠근다.
# -v /app/temp # bind mount 되어진 /app경로가 ro 인대, /app/temp 경로는 쓰기도 필요하므로 익명볼륨 설정
feedback-app # image name

```

- copy vs bind mount
  - bind mount 만 하면 되는대 왜 copy를 할까?
    - 도커 이미지가 생성될때에는 호스트폴더의 스냅샷만 복사한다.(COPY) 
    - 서버에 배포시에는 이 스냅샷을 쓰게되므로, 바인드마운트를 할 필요가 굳이 없다.
    - 반면에 개발환경에서는 실시간 변경사항을 컨테이너에 반영할 필요가 있으므로, 바인드마운트 를 사용한다.

- env (work run time) and arg (work build time)
  - docker run -d -p 3000:8000 --env PORT=8000
  - docker run -d -p 3000:8000 --env-file .env
  - docker build -t feedback-app --build-arg DEFAULT_PORT=8000 .

## docker networks
<img width="893" alt="스크린샷 2022-09-25 오후 2 25 44" src="https://user-images.githubusercontent.com/73451727/192129668-618e5558-1bbc-4079-8b3b-35a7cf6cdd4b.png">


- 방식 1. 컨테이너 내부에서 외부 인터넷과의 통신 (like www - api call)
	- 특별한 조작 없이 잘 통신됨
- 방식2. 컨테이너 내부에서 호스트 머신의 서비스와의 통신 (like db)
	- 도커가 인식할수 있는 특수도메인 필요 - localhost -> host.docker.internal 로 변경해야함
  - 도커가 자동변환 fetch(host.docker.internal).then(host machine ip)

- 방식3. 컨테이너와 컨테이너와의 통신(다중 컨테이너)
  - 1) 일반적인 방식 - 컨테이너 네트워크 
    - 도커가 인식할수 있는 특수도메인 필요(컨테이너 이름을 도메인으로 사용 가능) + 같은 도커 네트워크
		- 도커가 자동변환 fetch(container name:port/blabla).then(container ip)

    ```
		- docker network create my_net_1
		- docker network ls
		- docker run -d —name mongodb —network my_net_1
    ```
    
  - 2) 부가적인 방식 - IP 강제 맵핑
    - docker container inspect 
    - ipaddress 따서 localhost 대신에 연결시켜줌 
	  - 하드코딩 / 매번 바뀌기때문에 일반적인 방식은 아님
	  
## multiple container application (nodejs + react + mongodb)
<img width="885" alt="스크린샷 2022-09-25 오후 2 48 17" src="https://user-images.githubusercontent.com/73451727/192130265-535f14ee-89d1-4277-a0c6-2de1b8ca1726.png">

## DOCKER 컨테이너 배포하기 <br/>
- 개발 환경에서는 바인드마운트를 사용함 <br/>
실시간으로 수정되는 소스코드를 컨테이너에 반영할 필요가 있기때문에 <br/>
- 운영 환경에서는 바인드마운트를 사용하지않음 <br/>
그냥 COPY만 일어날 뿐임, 변경시에는 컨테이너를 새로 올림 <br/>
- DOCKERFILE 자체는 개발과 운영이 동일함 <br/>
  dockerfile에 바인드마운트를 하게되면 개발환경과 운영환경이 달라지므로 volume 설정이 docker run process에 존재하는 이유임 <br/>
- 이렇게 하면, 컨테이너를 실행하는 위치에 관계없이 동일해짐 <br/>

## ECS 배포
- docker-compose 는 로컬환경에서 사용하기 좋지만, production 환경에서 사용하기는 부적합함 <br/>
- docker-compose의 장점인, 여러개 컨테이너를 묶는 도커 네트워크 또한 사용하기 어려워짐 <br/>
- 그러나, aws ecs의 경우 동일한 테스크에 컨테이너를 추가하면 하나의 호스트 머신에 같이 올라감 <br/>
- 도커 네트워크 구성은 x / 단지 localhost를 사용할 수 있게 해줌 / mongodb:27017 -> localhost:27017 로 변경 <br/>
- 로컬환경 에서의 변수와 실제 배포시의 변수가 mongodb vs localhost <br/>
- ecr container 추가시에 variables를 추가로 설정한다 <br/>
- MONGODB_URL = localhost <br/>
- 도커화된 앱을 배포할 때는, 도커 네트워크를 사용할 수 없음(즉, 컨테이너 이름 자체로 다른 컨테이너에 접근하는것이 불가능해짐) <br/>
<br/>

- local에서처럼 db 볼륨을 이용하는 방법 -> EFS (elastic file system) <br/>
- 보안그룹설정 및 container edit -> storage 설정 필요 <br/>
- 롤링 배포시, 현재 컨테이너와 배포하려는 컨테이너가 동시에 EFS 접근하여 충돌함 - RDS와 같은 관리형 DB를 쓰자. <br/>
<br/>


