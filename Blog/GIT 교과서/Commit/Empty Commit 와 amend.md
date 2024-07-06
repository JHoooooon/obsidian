
---

커밋을 할때는 반드시 `commit message` 를 작성해야 하는 상황이 있다.

>[!info] 커밋 할때 커밋 메시지를 같이 작성해야 한다.

이러한 특별한 상황에 사용하는 옵션은 다음과 같다

```sh
$ git commit --allow-empty-message -m ""
```

이후 빈 커밋을 확인하면 다음처럼 나온다.

> [깃교과서 4.7.3](https://thebook.io/080212/0160/) 의 내용이다

```sh
$ git log 로그 확인
commit aa92947d350db27b604d1351930d4f809f96886e (HEAD -> master)
Author: hojin <infohojin@gmail.com>

Date:   Sat Jan 5 20:09:48 2019 +0900
 
commit aa1dd51a8883b2ea9a54209a00f434a2da01ee85
Author: hojin <infohojin@gmail.com>
Date:   Sat Jan 5 19:31:46 2019 +0900
    hello git world 추가
 
commit e2bce41380691b0a34aeab7db889a6c30fed8287
Author: hojin <infohojin@gmail.com>
Date:   Sat Jan 5 18:24:50 2019 +0900
	인덱스 페이지 레이아웃
```

빈 커밋인것을 볼 수 있다.

만약 `commit` 이후에, `message` 를 수정해야 하는 상황이면 어떻게 해야 할까?

그럴때 사용하는 명령어가 `amend` 이다.

```sh
$ git init;
$ touch test.txt;
$ echo "test" > test.txt;
$ git add test.txt;
$ git commit -am "init";
$ git commit --amend;
```

하면 다음의 커밋 메시지를 에디터 창에 표시한다

```sh
init --> 수정

# 변경 사항에 대한 커밋 메시지를 입력하십시오. '#' 문자로 시작하는
# 줄은 무시되고, 메시지를 입력하지 않으면 커밋이 중지됩니다.
#
# 시각:      Sat Jul 0 00:00:00 0000 +0900
#
# 현재 브랜치 master
#
# 최초 커밋
#
# 커밋할 변경 사항:
#       새 파일:       test.txt
#
```

`vim` 에디터를 사용해서 `message` 를 수정해주고, `:wq!`  를 하면, `vim` 저장되고 닫힌다.

```sh
$ git log

commit 82c3ad67b519c8db5f09e9a10d0d76725b191580 (HEAD -> master)
Author: jhoon <jaehoon0822@gmail.com>
Date:   Sat Jul 6 22:31:33 2024 +0900

    수정
```

`commit` 메시지가 수정된것을 볼 수 있다.