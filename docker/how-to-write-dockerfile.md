# How to write Dockerfile

## 중복설정하기

[Don't Repeat Yourself with Anchors, Aliases and Extensions in Docker Compose Files](https://medium.com/@kinghuang/docker-compose-anchors-aliases-extensions-a1e4105d70bd)

설정을 중복 사용하기 위한 곳에 아래와 같이 설정한다.
```
...
  ...: &nameyouwant
```

사용은 아래와 같이 한다.
```
...
  ...:
    <<: *nameyouwant
```
