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
  
  도커 이미지가 생성될때에는 호스트폴더의 스냅샷만 복사한다. 서버에 배포하는 환경에서는 실시간으로 변경사항이 컨테이너에 반영될 필요가 없지만 개발환경에서는 실시간 변경사항을 컨테이너에 반영할 필요가 있다. 이럴때 유용한것이 바인드마운트 이다.

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




