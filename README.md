# docker-k8s

## docker volume concept
<img width="939" alt="스크린샷 2022-09-06 오전 9 16 09" src="https://user-images.githubusercontent.com/73451727/188522927-03baca1a-93d3-45d4-a6b0-c3fd7dc10083.png">
  
- application: read only
  
- temporary app data: read + write in container
  
- permanent app data: read + write in container and volume


```
# data-volumes-02-added-dockerfile
docker build . -t docker-volume-test-01
docker run -d -p 3000:80 --rm docker0-volume-test-01
```
