
>[!info] `개발자를 위한 레디스` 의 내용과 `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`Geo API` 는 `Redis` 에서 공식적으로 소개한 `API` 이다.
이는 지리위치 관련 저장 및 쿼리을 지원한다.

`GEOADD` 로 경도, 위도를 추가할때, `Redis` 는 내부적으로 $52bit$ `GEOHASH` 로 변환한다.
이는 일반적으로 허용하는 `GEO encoding` 시스템이다.

저장된 `GEO` 와 `GEOPOS` 가 반환하는 좌표간에는 약간의 차이가 있음을 고려하는것이 좋다.
그래서 완벽하게 $2$ 개가 

---

다음은 `GEOADD` 를 사용하여 경도, 위도 데이터 쌍을 집합으로 지리 데이터를 저장한다.

>[!info] GEOADD
```sh
127.0.0.1:6379> GEOADD restaurants:CA -121.896321 37.916750 "Olive Garden" -117.910937 33.804047 "P.F. Chang's" -118.508020 34.453276 "Outback Steakhouse" -119.152439 34.264558 "Red Lobster" -122.276909 39.458300 "Longhorn Charcoal Pit" 
(integer) 5
```

저장한 경도, 위도 쌍을 쿼리하려면 `GEOPOS` 명령어를 사용한다.

>[!info] GEOPOS
```sh
127.0.0.1:6379> GEOPOS restaurants:CA "Red Lobster" 
1) 1) "-119.1524389386177063" 
   2) "34.26455707283378871" 
```

현재 `location` 에서 $5km$  내의 `member` 를 알고싶다면, `GEORADIUS` 명령어를 사용한다.

>[!note] 책에서는 내 위치가 `Mount Diablo` 주립공원에 있고(`-121.923170 37.878506`), 근방 $5km$ 내의 레스토랑을 찾는다는 가정이다.

>[!info] GEORADIUS
```sh
127.0.0.1:6379> GEORADIUS  restaurants:CA  -121.923170 37.878506 5 km 
1) "Olive Garden" 
```

다음은 두 `member` 의 거리를 비교하여 거리를 반환한다.
다음은, `km` 단위로 두 `memeber` 의 사이를 반환한다.

>[!info] GEODIST
```sh
127.0.0.1:6379> GEODIST restaurants:CA "P.F. Chang's" "Outback Steakhouse" km 
"90.7557"
```

만약, `member` 의 `location` 을 기준으로, 주변 $100km$ 내의 `member` 를 알고 싶다면, `GEORADIUSBYMEMBER` 를 사용한다.

>[!info] GEORADIUSBYMEMBER
```sh
127.0.0.1:6379> GEORADIUSBYMEMBER restaurants:CA "Outback Steakhouse" 100 km 
1) "Red Lobster" 
2) "Outback Steakhouse" 
3) "P.F. Chang's"
```

