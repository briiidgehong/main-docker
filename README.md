# docker-k8s

## docker volume concept
  ![image.jpg1](https://user-images.githubusercontent.com/73451727/188522927-03baca1a-93d3-45d4-a6b0-c3fd7dc10083.png) |![image.jpg2](https://user-images.githubusercontent.com/73451727/189511780-b505af93-d771-42c9-a7fb-3687d984ff83.png)
--- | --- | 

- application: in image / read only
  
- temporary app data: in container / read + write / 컨테이너 삭제시 같이 삭제됨
  
- permanent app data: in container and volume / read + write / 컨테이너가 삭제되어도 보존 가능


```
# data-volumes-02-added-dockerfile
docker build . -t docker-volume-test-01
docker run -d -p 3000:80 --rm docker0-volume-test-01
```
