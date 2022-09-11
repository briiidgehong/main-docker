# docker-k8s

## docker volume concept
  ![image.jpg1](https://user-images.githubusercontent.com/73451727/188522927-03baca1a-93d3-45d4-a6b0-c3fd7dc10083.png) |![image.jpg2](https://user-images.githubusercontent.com/73451727/189511780-b505af93-d771-42c9-a7fb-3687d984ff83.png)
--- | --- | 

- application: read only
  
- temporary app data: read + write in container
  
- permanent app data: read + write in container and volume


```
# data-volumes-02-added-dockerfile
docker build . -t docker-volume-test-01
docker run -d -p 3000:80 --rm docker0-volume-test-01
```
