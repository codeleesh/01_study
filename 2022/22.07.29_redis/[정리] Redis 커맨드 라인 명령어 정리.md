





ex) SET key value [ NX | XX ] [GET] [ EX seconds | PX milliseconds | EXAT unix-time-seconds | PXAT unix-time-milliseconds | KEEPTTL]

- 필수값은 key, value 입니다.

```
> SET USERS:1 lsh
OK
```

- 값의 경우 겹쳐쓰기가 가능하나, `NX` 옵션을 사용하면 겹쳐쓰기 방지가 가능합니다.

```
> SET USERS:1 lsh
OK
> SET USERS:1 kkk NX
(nil)
```

- `XX` 옵션은 지정한 키가 이미 저장이 되어 있어야만 저장이 가능합니다.

```
> SET USERS:2 lsh XX
(nil)
```

- `EX`, `PX`, `EXAT`, `PXAT` 등 을 설정하여서 데이터 만료 시간을 설정할 수 있습니다.

```
> SET USERS:2 lsh EX 5
OK
> GET USERS:2
"lsh"
> GET USERS:2
(nil)
```

