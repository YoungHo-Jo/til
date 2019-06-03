# How to write Dockerfile

## 중복설정하기

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
