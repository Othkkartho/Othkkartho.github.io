---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(9)-셸 스크립트 프로그래밍

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: Ubuntu 서버 구축
# multiple tag entries are possible
tags: [Linux, Ubuntu 서버 구축]
# thumbnail image for post
img: ":/linux/ubuntu/ubuntu.avif"
# disable comments on this page
#comments_disable: true

# publish date
date: 2023-06-30 22:00:00 +0900

# seo
# if not specified, date will be used.
# meta_modify_date: 2023-04-14 17:00:00 +0900
# check the meta_common_description in _data/owner/[language].yml
#meta_description: ""

# optional
# if you enabled image_viewer_posts you don't need to enable this. This is only if image_viewer_posts = false
#image_viewer_on: true
# if you enabled image_lazy_loader_posts you don't need to enable this. This is only if image_lazy_loader_posts = false
#image_lazy_loader_on: true
# exclude from on site search
#on_site_search_exclude: true
# exclude from search engines
#search_engine_exclude: true
# to disable this page, simply set published: false or delete this file
# published: false
---

<!-- outline-start -->

셸의 개념부터 셸 스크립트 프로그래밍까지를 공부해본다.

<!-- outline-end -->

* * *

### 목차

1. [셸의 개념과 특징](#셸의-개념과-특징)
2. [셸 스크립트 프로그래밍](#셸-스크립트-프로그래밍)

### 셸의 개념과 특징
리눅스 셸은명령과 프로그램을 실행할 때 사용하는 인터페이스로 사용자가 입력한 명령을 해석해 커널에 전달하거나 커널의 처리 결과를 사용자에계 전달하는 역할을 합니다.  
우분투에서 기본적으로 사용하는 셸은 bash였으나 현재 우분트를 보게 되면 dash로 변경되었음을 확인할 수 있습니다.  \n

**셸의 기능 비교**{:data-align="center"}

| 기능 | sh | csh | ksh | bash | tcsh | zsh |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| Job Control | X | O | O | O | O | O |
| Aliases | X | O | O | O | O | O |
| Command History | X | O | O | O | O | O |
| Command Line Editing | X | X | O | O | O | O |
| Login/Logout Watching | X | X | X | X | O | O |
| File Name Completion | X | O | O | O | O | O |
| Spelling Correction | X | X | X | X | O | O |
| Process Substitution | X | X | X | O | X | O |
| Shell Scripts | O | O | O | O | O | O |
| Freely Available | X | X | X | O | O | O |

각 기능들의 설명은 [여기로](#기능-설명), 위 종류 Shell들의 자세한 설명이 보고싶다면 [여기로](#다양한-유형의-shell)

#### 환경변수
- `export 환경변수=값` 을 실행하면 환경 변수 값을 변경할 수 있고, `printenv`를 실행하면 현재 환경 변수들과 그 환경변수의 값을 알 수 있습니다.
- 설정된 환경 변수는 echo $환경변수 명령으로 확인할 수 있습니다.

### 셸 스크립트 프로그래밍
#### 셸 스크립트 작성
- `vi name.sh` 와 같이 .sh파일을 만들고, 안에 스크립트를 작성합니다.

#### 셸 스크립트 실행
- `sh 스크립트 파일`로 셸 스크립트를 실행합니다.
- `chmod +x 파일명`으로 파일 권한에 실행을 추가한 다음 `./스크립트파일`을 실행하면 됩니다.

#### 변수
- myVar = Hello -> "=" 앞뒤에 공백이 있어 오류가 발생합니다.
- myVar=Hello
- myVar=Yes Sir -> 값에 공백이 있다면 " "으로 묶어야 합니다.
- myVar="Yes Sir"
- myVar=7+5 -> 정상이지만 7+5라는 문자열로 인식해 문자 그대로 출력됩니다.
- $가 포함된 글자를 출력하려면 ''로 묶거나 앞에 \를 넣어야합니다.
- " "로 변수를 묶거나 묶지 않아도 출력됩니다.
- **숫자 계산** 시에는 \`expr $num1 + 200\` 과 같이 \`로 반드시 묶고 expr 키워드를 사용해야 합니다. +, -, /와 달리 *도 예외적으로 앞에 \를 넣어야 합니다.
- 파라미터 변수의 경우 $0, $1과 같은 형태입니다. 만약 `sh paravar.sh 1번 2번`과 같은 명령어가 있다면 $0은 paravar.sh, $1은 '1번', $2는 '2번'입니다. $*는 '1번 2번'이 출력됩니다.

#### 조건문
**기본 if문**
```
if [ 조건 ]
then
    참인 경우 실행 명령어
fi
```

**if~else문**
```
if [ 조건 ]
then
    참인 경우 실행 명령어
else
    거짓인 경우 실행 명령어
fi
```

**case문**
```
case "$1 in
    start)
        echo "시작";;
    stop)
        echo "중지";;
    restart)
        echo "다시 시작";;
    *)
        echo "다시 입력";;
```
  
and의 경우 -a 또는 &&를, or은 -o 또는 ||를 사용합니다.

#### 반복문
**for ~ in 문**
```
#!/bin/sh
hap=0
for i in 1 2 3 4 5
do
    hap=`expr $hap + $i`
done
echo "1부터 5까지의 합: "$hap
exit 0
```

**while문**
```
#!/bin/sh
while [ 1 ]
do
    echo "우분투 22.04 LTS"
done
exit 0
```

- until 문: while문과 용도가 거의 같지만 조건식이 참일 때까지 계속 반복 실행합니다.
- break 문: 반복문을 종료할 때 주로 사용합니다.
- continue 문: 반복문의 조건식으로 돌아가게 합니다.
- exit 문: 해당 프로그램을 완전히 종료합니다.
- return 문: 함수를 호출한 곳으로 돌아가게 합니다.

#### 셸 스크립트 응용 기능
- 사용자 정의 함수
```
함수명 () { <- 함수 정의
    내용
}
함수명 <- 함수 호출
```

- eval: 문자열을 명령문으로 인식하여 실행합니다.
- export: 특정 변수를 전역 변수로 만들어 모든 셸에서 사용할 수 있게 합니다.
- printf: C언어의 printf()함수와 비슷하게 형식을 지정하여 출력합니다.
- set과 $(명령): 결과를 파라미터로 사용하려면 set 명령을 이용해야 합니다.
```
set $(date)
echo "요늘은 $4 요일 입니다."
```
- shift: 파라미터 변수를 왼쪽으로 한 단계씩 아래로 시프트 시킵니다.

### 참고
#### 다양한 유형의 Shell
1. 본 셸 (sh)
    - 컴팩트한 특성과 빠른 동작 속도로 인기를 얻었습니다.
    - 주요 단점으로는 논리 및 산술 연산을 처리하는 기능이 내장되어 있지 않습니다.
    - Linux 대부분의 다른 유형의 셸과 달리 이전에 사용된 명령을 편하게 재사용할 수 없습니다.
    - 적절한 대화형 사용을 제공하기 위한 포괄적인 기능이 부족합니다.
2. C셸 (csh)
    - 산술 연산에 대한 내장 지원 및 C 프로그래밍 언어와 유사한 구문과 같은 유용한 프로그래밍 기능을 포함하도록 개발되었습니다.
    - 히스토리와 별명(alias), 작업 제어 기능도 대표적인 기능으로 있습니다.
    - 히스토리는 과거에 사용한 명령어를 반복하거나 수정하기 편리하고, 별명은 자주 쓰는 긴 명령어를 짧게 사용할 수 있도록 도와주었으며, 작업 제어는 프로세서에 우선순위를 두는 것으로 효율적인 작업이 가능하도록 하였습니다.
3. ksh
    - 본셸과 하위 호환되며, C셸의 수많은 기능을 포함합니다.
4. GNU Bourne-Again 셸 (bash)
    - Bourne Shell과 호환되도록 설계되었습니다. ksh 및 csh에서 많은 아이디어를 받아 명령 히스토리, 디렉터리 스택, $RANDOM POSIX 형식 명령어 치환 등을 지원합니다.
    - 입력 중에 명령어나 파일 이름을 자동 완성해 주는 기능도 지원합니다.
5. tcsh
    - csh기반이면서 csh과 호환되는 유닉스 셸
    - 명령 줄 완성, 명령 줄 편집등의 기능이 포함된 csh입니다.
    - 다른 셸들과 달리 스크립트 안에 함수를 정의할 수 없으며 사용자는 csh에서처럼 별칭을 대신 사용해야 합니다.
6. zsh
    - 상호작용 로그인 셸이자 셸 스크립트를 위한 강력한 명령줄 인터프리터로 사용할 수 있습니다.
    - bash, ksh, tcsh의 일부 기능을 포함애 수많은 개선사항이 갖춰진 확장형 본셸입니다.
    - 커스텀이 비교적 자유로운 편이며, 설치 또한 간편합니다.


#### 기능 설명
- Job Control: 작업 제어는 사용자가 백그라운드에서 프로세스를 시작하거나, 이미 실행중인 프로세스를 백그라운드로 보내고, 백그라운드 프로세스를 포그라운드로 가져오고, 프로세스를 일시 중단하거나 종료할 수 있도록 하여 이를 가능하게 하기 위해 개발된 기능입니다.
- Aliases: 시스템 명령어를 단축시키기 위해 주로 사용되며, 그 외의 주기적으로 사용되는 명령어에 기본 변수를 추가하기 위해 사용됩니다.
- Command History: 사용했던 명령어를 저장하고, 사용할 수 있도록 하는 기능입니다.
- Command Line Editing: 명령어를 잘못 입력했을 때 수정할 수 있는 기능입니다.
- Login/Logout Watching: ??
- File Name Completion: 파일 이름 자동 완성
- Spelling Correction: 맞춤법(철자) 수정
- Process Substitution: 프로세스 치환은 프로세스의 출력을 다른 프로세스에 넣어주는 것을 의미한다. 즉 리디렉션(표준 입출력의 방향을 바꾸는 것)을 말합니다.
- Shell Scripts: 셸 프로그래밍(스크립트)를 할 수 있습니다.
- Freely Available: ??

### 참고 자료
1. [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)
2. [DigitalOcean - What are the Different Types of Shells in Linux?](https://www.digitalocean.com/community/tutorials/different-types-of-shells-in-linux)
3. [위키백과 - 본 셸](https://ko.wikipedia.org/wiki/%EB%B3%B8_%EC%85%B8)
4. [위키백과 - 배시(유닉스 셸)](https://ko.wikipedia.org/wiki/%EB%B0%B0%EC%8B%9C_(%EC%9C%A0%EB%8B%89%EC%8A%A4_%EC%85%B8))
5. [위키백과 - C 셸](https://ko.wikipedia.org/wiki/C_%EC%85%B8)
6. [위키백과 - tcsh](https://ko.wikipedia.org/wiki/Tcsh)
7. [위키백과 - 콘셸](https://ko.wikipedia.org/wiki/%EC%BD%98%EC%85%B8)
8. [위키백과 - Z 셸](https://ko.wikipedia.org/wiki/Z_%EC%85%B8)