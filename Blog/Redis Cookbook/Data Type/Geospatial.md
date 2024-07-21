
>[!info] `개발자를 위한 레디스` 의 내용과 `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`Geo API` 는 `Redis` 에서 공식적으로 소개한 `API` 이다.
이는 지리위치 관련 저장 및 쿼리을 지원한다.

`GEOADD` 로 경도, 위도를 추가할때, `Redis` 는 내부적으로 $52bit$ `GEOHASH` 로 변환한다.
이는 일반적으로 허용하는 `GEO encoding` 시스템이다.

저장된 `GEO` 와 `GEOPOS` 가 반환하는 좌표간에는 약간의 차이가 있음을 고려하는것이 좋다.
그래서 정확하게 $2$ 개가 동일한것이라고 기대해서는 안된다

`GEORADIUS` 와 `GEORADIUSBYMEMBER` 명령어에서, `WITHDIST` 옵션을 사용할수 있다.
`ASC/DESC` 옵션은 내림차 혹은 오름차 순서로 정렬하여 보여준다.

>[!warning] `GEORADIUS` 와 `GEORADIUSBYMEMBER` 는 `deprecated` 되었다.

>[!info] `GEOSEARCH` 는 $O(N + log(M))$ 으로 $N$ 은 중심에서의 반경으로 만들어진 원형 영역 경계에서 `box` 요소수이다.<br><br>따라서 뛰어난 성능을 원한다면 하나의 쿼리에서 반경 매개변수를 가능한 작게 설정하여 더 적은 포인트를 포함해야 한다.

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

---

>[!warning] `GEORADIUS` 와 `GEORADIUSBYMEMBER` 는 `deprecated` 되었다.<br>다음은 `Redis` 에서 사용을 권장하는 `GEOSEARCH` 의 예제이다.<br><br>[GEOSEARCH](https://redis.io/docs/latest/commands/geosearch/) 및 [GEOSEARCHSTORE](https://redis.io/docs/latest/commands/geosearchstore/)에서 확인가능하다.


>[!info] GEOSEARCH
```sh
GEOSEARCH key <FROMMEMBER member | FROMLONLAT longitude latitude>
  <BYRADIUS radius <m | km | ft | mi> | BYBOX width height <m | km |
  ft | mi>> [ASC | DESC] [COUNT count [ANY]] [WITHCOORD] [WITHDIST]
  [WITHHASH]
```

`query` 시 `center` 위치를 제공하는 **필수옵션**들로 이중 하나만 제공가능하다.

- **FROMMEMBER**: 주어진 `member` 의 `position` 을 기반으로 `search` 한다
- **FROMLONLAT**: 주어진 `longitude`, `latitude` 위치를 기반으로 `search` 한다.

`query` 시 `shape` 를 제공하는 **필수옵션**들로 이중 하나만 제공가능하다.

- **BYRADIUS**: `GEOREDIUS` 와 비슷하며, 주어진 `radius` 에 따라 원형 영역 내부를 검색한다.
- **BYBOX**: `height` 과 `width` 에 따라 결정되며, 축 정렬 직사각형 내부를 검색한다.

`query` 시 추가적인 정보를 제공하는 **선택적 옵션**들이다.

- **WITHDIST**: `member` 들의 지정된 중심점에서 부터 거리를 반환한다.<br> 거리는 `redius` 혹은 `width`, `height` 인자로 지정된 단위와 동일한 단위로 리턴된다.
- **WITHCOORD**: 매칭되는 `member` 들의 경도, 위도를 리턴한다.
- **WITHHASH**: `member` 의 `GEOHASH` 로 `encoding` 된 `raw` 데이터를 정렬된 집합으로 반환한다.<br>이는 저수준 단계에서 해킹 또는 디버깅하는데 유용하며, 일반 사용자에게는 별 관심이 없는 옵션이다.

`query` 시 매칭된 `member` 는 기본적으로 정렬되지 않은 값이다.
만약 정렬을 하고 싶다면 다음의 선택적 옵션중 하나를 사용할수 있다.

- **ASC**: 중심점으로 부터 가장 가까운 항목부터 먼 항목까지 정렬된다
- **DIST**: 중심점으로 부터 가장 먼 항목부터 가까운 항목까지 정렬된다

`query` 시 매칭된 `member` 의 개수를 몇개 출력할지 선택하는 선택적 옵션이다.

 - **COUNT**: `count` 숫자만큼 `member` 위치에 가까운순으로 반환한다.
 - **COUNT count ANY**: `ANY` 를 사용하면 거리와 상관없이 `N` 개의 데이터를 반환한다.<br>정확도가 떨어지지만 빠르게 검색해야할 실시간 데이터같은경우 사용할수 있다고 한다.

>[!info] GEOSEARCHSTORE
```sh
GEOSEARCHSTORE destination source <FROMMEMBER member |
  FROMLONLAT longitude latitude> <BYRADIUS radius <m | km | ft | mi>
  | BYBOX width height <m | km | ft | mi>> [ASC | DESC] [COUNT count
  [ANY]] [STOREDIST]
```

`GEOSEARCHSTORE`  는 `GEOSEARCH` 와 비슷하지만, 그 결과를 `destination key` 에 저장한다.
약간 다른 옵션이 몇가지 더 있는데,  다음과 같다

- **destination**: 이는 `Store` 할 `목적지 key` 이름이다.
- **source**: 데이터를 가져올 `key` 이다.

`query` 시 선택적 옵션으로 저장방식을 선택할수 있다.

- **STOREDIST**: 이 명령은 `circle` 또는 `box` 의 중심으로 부터 거리인 `value` 와 `member` `key` 값을 정렬된 집합으로 저장한다. 이때, `value` 값은 부동소수점형식으로 변환되어 저장된다.

---



