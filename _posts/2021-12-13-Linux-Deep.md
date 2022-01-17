---
title: Linux 심화 - 사용권한, 환경변수
date: 2022-01-14 +0000
categories: Linux
tags: [Linux, CLI]
---

## 사용권한

### File Owner

리눅스에는 세 부류의 파일 소유자가 있다.

- Owner(혹은 User): 파일의 소유자(보통 파일을 만든 주체)
- Group: 여러 Owner(혹은 User)를 포함한다.
  프로젝트를 진행하면서, 파일에 많은 사람이 액세스해야 하는 경우, 각 user에게 일일이 권한을 할당하는 대신 모든 user를 그룹으로 묶고 그룹에 권한을 할당한다.
- Others: owner도 아니고 group도 아닌 소유자이다.
  소유자를 `other`로 설정하면 global한 설정이라고 볼 수 있다.

파일 소유자를 확인하는 방법:

```console
 DJM ☠️ ~/jinyoung/codestates/environment_variable$ ls -al
total 32
drwxr-xr-x   7 jin-yeonggim  staff  224 12 13 13:54 .
drwxr-xr-x@ 30 jin-yeonggim  staff  960 12 13 13:38 ..
-rw-r--r--   1 jin-yeonggim  staff   20 12 13 13:54 .env
-rw-r--r--   1 jin-yeonggim  staff   84 12 13 13:57 index.js
drwxr-xr-x   4 jin-yeonggim  staff  128 12 13 13:49 node_modules
-rw-r--r--   1 jin-yeonggim  staff  860 12 13 13:49 package-lock.json
-rw-r--r--   1 jin-yeonggim  staff  265 12 13 13:49 package.json
```

터미널에서 명령어 `ls` 뒤에 option으로 `-al`을 붙인다.
출력된 결과를 해석하면 다음과 같다.

```console
drwxr-xr-x   4 jin-yeonggim  staff  128 12 13 13:49
# node_modules
#    ↓                ↓          ↓
# Permission       Owners      Group
```

### File Permission

▸ 파일 권한을 나타내는 결과는 위와 같은 길이가 10인 문자열이다.

- 첫 문자는 File의 Type을 나타낸다.
- 다음 9개의 문자는 세 개씩 끊어 읽어야 한다.
  각각 owner/user의 파일 권한, group의 파일 권한, others의 파일 권한을 나타낸다.

```console
drwxr-xr-x
# d: 파일이냐 폴더냐 나타내는 자리
# rwx: owner/user의 파일 권한을 나타내는 자리
# r-x: group의 파일 권한을 나타내는 자리
# r-x:: others의 파일 권한을 나나태는 자리
```

▸ File Type은 다음과 같다.

| value |     File Type      |
| :---: | :----------------: |
|   -   |      일반파일      |
|   d   |   디렉토리 파일    |
|   b   | 블록 디바이스 파일 |
|   c   | 문자 디바이스 파일 |
|   l   |  심볼릭 링크 파일  |

이 중에서 주로 `-` 와 `d` 를 사용한다.

▸ 권한을 나타내는 세 개의 상태는 다음과 같다.

| 문자 |        의미         |
| :--: | :-----------------: |
|  r   |  read - 읽기 권한   |
|  w   |  write - 쓰기 권한  |
|  x   | execute - 실행 권한 |

`r`, `w`, `x`이면 해당 권한이 있음을 나타내고, `-` 이면 해당 권한이 없음을 나타낸다.

### 권한 변경 (Change File Permission)

Linux 의 shell 에서 제공하는 `chmod` 명령어를 사용하여 파일의 권한을 변경할 수 있다.
두 가지 방식으로 변경할 수 있다.

#### Symbolic method

`+` 와 `-` , `=` 을 이용한 방법이다.

|  Access Class   | Operator            | Access Type |
| :-------------: | :------------------ | :---------- |
|     u(user)     | +(add access)       | r(read)     |
|    g(group)     | -(remove access)    | w(write)    |
|    o(other)     | =(set exact access) | x(execute)  |
| a(all: u, g, 0) |                     |

예시:

```console
chmod a+r filename # 모든 사용자에게 읽기 권한 부여
chmod +r filename # access class를 생략하면 기본값 all 적용
chmod go-rw filename # group 과 others 에게서 읽기, 쓰기 권한을 제거
chmod go=r filename # group 과 others 는 읽기 권한만 가진다.
chmod o-w dirname # other 에게서 디렉터리 write 권한을 제거한다. rm 등을 사용할 수 없게 된다.
```

#### Absolute method

`rwx` 를 이진수로 해석해 세자리 숫자로 표기하는 방법이다.

| 숫자 |        계산        | 문자표기 |          의미           |
| :--: | :----------------: | :------: | :---------------------: |
|  7   | 4(r) + 2(w) + 1(x) |   rwx    | read, write and execute |
|  6   | 4(r) + 2(w) + 0(-) |   rw-    |     read and write      |
|  5   | 4(r) + 0(-) + 1(x) |   r-x    |    read and execute     |
|  4   | 4(r) + 0(-) + 0(-) |   r--    |        read only        |
|  3   | 0(-) + 2(w) + 1(x) |   -wx    |    write and execute    |
|  2   | 0(-) + 2(w) + 0(-) |   -w-    |       write only        |
|  1   | 0(-) + 0(-) + 1(x) |   --x    |      execute only       |
|  0   | 0(-) + 0(-) + 0(-) |   ---    |          none           |

위 표에 케이스를 조합해 세자리 숫자로 사용 권한을 설정할 수 있다.

```console
chmod 744 myfile
# 744 → rwxr--r--
```

추가적인 내용은 [링크](https://kb.iu.edu/d/abdb)를 통해 확인할 수 있다.

### sudo 와 su

`sudo` 명령어는 다른 `user` (기본적으로는 `superuser - root`) 보안 특권으로 프로그램을 실행할 수 있도록 한다. `sudo` 를 사용하면 `user` 나 `group` 이 `root password` 를 몰라도 몇몇(혹은 모든) 명령을 사용할 수 있게 된다.

```console
sudo command
```

`sudo` 를 사용하면 [principle of least privilege](https://kb.iu.edu/d/amsv)(PoLP)를 쉽게 구현할 수 있다.

`su` 는 `switch user` 의 줄임말이다.

```console
su user
```

`su` 뒤에 전환하고자 하는 계정 명을 쓴다.
생략할 수도 있는데, 이 경우 *시스템 관리자 계정*인 `root` 로 설정된다.
`su` 명령을 실행하면 전환하려고 하는 계정의 `password` 입력을 요구받게 된다.
맥을 사용할 때 초기에는 `root` 에 `password` 가 설정되어 있지 않다.

<details>
<summary>`root` 에 비밀번호를 설정하는 방법</summary>
<div markdown="1">

```console
sudo -s
```

를 입력하고 `password` 입력하라고 뜨면 현재 로그인 되어 있는 사용자 계정 비밀번호(맥북 부팅할 때 입력하는 비밀번호)를 입력한다.

```console
whoami
root
```

`whoami` 를 통해 현재 계정이 `root` 임을 확인할 수 있다.

```console
passwd root
```

를 입력하면 `password`를 입력하라고 뜨는데 여기에 `root` 비밀번호를 입력한다.
한 번 더 입력하라고 하면 다시 입력해준다.

</div>
</details>

`su` 명령어를 사용하는 것은 추천되지 않는데 `root` 계정으로 로그인하게 되면 시스템을 마음대로 바꿀 수 있기 때문에 예기치 못한 문제를 일으킬 수도 있기 때문이다.
`root` 계정으로 로그인했다가 빠져나오려면 `exit` 명령어를 사용한다.

`sudo` 와 `su` 의 차이

- `sudo` 는 `password` 를 요구 받지 않고 명령을 한 번 실행한다.
- `su` 는 사용자를 `root` 로 바꾸어버린다.

`admin` 과 `user`, `root` 사이 관계

- `root` 는 시스템에 대한 모든 권한을 가지고 있는 `superuser` 이다.
- `user` 는 일반 유저
- `admin` 도 `user` 이긴 한데 `root` 로부터 시스템 환경에 접근 및 수정할 수 있는 권한을 부여받은 유저이다. 사용자들이 시스템 안에서 하게 되는 일반적인 작업들(파일을 만들고, 프로그램을 돌리고, ...)은 `user` 계정으로 실행하고 `system` 관련한 작업들(OS 환경설정 등)은 `admin` 이라는 계정을 통해 하도록 구분지은 것 뿐이다.

linux를 잘 알면 좋은 점

- 업무 자동화 유리
- aws 사용할 때 유리

## 환경변수

### UNIX 환경변수

rough하게 정의하면 운영체계가 굴러가는데 필요한 변수들의 모임이다.
시스템에 정의된 환경변수를 확인하려면 터미널에 다음 명령어를 입력한다.

```console
export
# printenv로도 조회가능하다
# env로도 조회가능하다
```

아무런 인자 없이 위와 같이 입력하면 OS 에 설정된 모든 환경변수들의 목록을 볼 수 있다.
특정 환경변수의 값만 확인하고 싶을 때에는 `echo` 명령어를 사용한다.

```console
echo $환경변수명
# echo 명령어를 사용할 때에는 환경변수명 앞에 $ 기호를 붙여야 한다.

# printevn 명령어를 사용할 때에는 아래와 같이 한다.
# printenv 사용할 때에는 변수명 앞에 $를 쓰지 않아도 된다.
printenv 환경변수명

# env로 특정 환경변수를 조회할 때에는 아래와 같이 grep을 사용한다.

env | grep 환경변수명

# 환경변수 이름에 공백이 있는 경우 따옴표로 감싸준다.
```

```console
echo $LANG
# echo 명령어로 현재 쉘의 환경변수 뿐 아니라 시스템 전역변수도 읽을 수 있다.
# 시스템 전역변수를 읽는 방법으로 printenv name 도 있다.
```

을 입력하면

```console
ko_KR.UTF-8
```

와 같은 결과를 얻게 된다.
`export` 를 사용해 <u>현재 터미널에서만 사용가능한</u> 환경변수를 설정할 수도 있다.

```console
export name=value
```

예를 들어

```console
export NickName='DongDong'
```

이라고 설정할 수 있다.
다만 이렇게 설정한 환경변수는 현재의 터미널에서만 사용가능하다.
이렇게 설정한 환경변수를 터미널을 끄거나 시스템을 재부팅하면 삭제된다.

`bash` 쉘 변수를 특정 유저가 영구적으로 사용하도록 만들기 위해서는 `~/.bashrc` 파일에 설정해줘야 한다. (실제 터미널에서 실험해봤는데 새롭게 설정한 변수가 초기화된다. 이유 확인 필요)
`bash` 쉘 변수를 모든 유저가 영구적으로 사용하도록 만들기 위해서는 `/etc/bash.bashrc` 파일에 해당 변수를 추가 해줘야 한다. (이 경우에도 변수가 자꾸 초기화된다. 이유가 뭘까?)

### Node.js에서 환경변수 설정하기

[npm의 dotenv 모듈](https://www.npmjs.com/package/dotenv#Config)을 사용해 자바 스크립트에서 환경변수를 사용할 수 있다.
아래와 같이 간단하게 `node.js`를 이용해 환경변수를 확인해보자.

```console
# terminal
~$mkdir environment-test
~$cd environment-test
~/environment-test$npm init
...
# 여러 번 Enter를 치면서 node.js 프로젝트 초기설정을 해준다.
...
# 자바 스크립트 파일을 만든다.
~/environment-test$nano index.js
# nano 편잡창에 js 코드를 작성하고 저장한다.

# 작성한 내용 확인한다.
~/environment-test$cat index.js
console.log(process.env);
~/environment-test$node index.js
# 환경변수를 담은 배열이 터미널에 출력될 것이다.
```

`node` 프로젝트에 포한되어 있는 내장 객체 [process.env](https://nodejs.org/docs/latest/api/process.html#process_process_env) 를 이용하면 시스템 환경변수에 접근할 수 있다.
터미널에서 `export` 명령을 입력했을 때와 동일한 결과를 볼 수 있다.
특정 환경변수을 조회하는 것도 가능하다.

```js
console.log(process.env.LANG); // ko_KR.UTF-8
// process.env 객체를 사용하는 것은 node.js를 이용하는 프로젝트에 기본적으로 포함된 것으로 보인다.
// 즉, npm init로 생성한 프로젝트에서 process.env를 사용할 수 있는 것 같다.
```

`node.js` 환경에서 시스템 환경변수를 새롭게 설정하기 위해서는 [dotenv 모듈](https://www.npmjs.com/package/dotenv)을 사용해야 한다.

```console
npm i dotenv // 설치
```

`dotenv 모듈`은 사용자가 작성한 `.env 파일`을 `process.env` 객체에 적용할 수 있도록 해준다.
`.env` 파일을 만들어 거기에 필요한 환경변수를 작성하고 나서 `dotenv.config();` 를 생행하면 `.env` 파일에 정의된 환경변수가 `process.env` 객체에 추가된다.

```console
~/environment-test$nano .env
# 환경변수 작성
...
# 작성 내용 확인
~/environment-test$cat .env
myname=dongdong

# 작성한 .env 파일을 process.env 객체에 추가하기 위한 js파일 작성
~/environment-test$nano index.js
...
# 작성한 index.js 파일 내용 확인
~/environment-test$cat index.js
const dotenv = require('dotenv');
dotenv.config();
console.log(process.env.myname);

# myname 이라는 환경변수가 설정됐는지 확인하기
~/environment-test$node index.js
dongdong
```

`.env` 파일은 `환경변수이름=값` 의 형태로 작성해야 한다.
`'='` 기호 앞뒤에는 공백이 없으며, 텍스트를 감싸는 어떠한 괄호도 없다.

### 주의할 점

`.env` 파일을 `git`에 올리면 안된다.
제외 시키기 위해 `git.ignore`파일에 `.env`를 써줘야 한다.
예를 들어 DB Password, 혹은 API Key의 값을 담고 있는 `.env` 파일을 웹에 올리게 되면 보안이 무너질 뿐 아니라, 해커가 심어놓은 봇을 통해 내 활동이 그대로 report될 수도 있다.
