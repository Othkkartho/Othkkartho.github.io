---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(5)-리눅스 사용자 관리와 파일 관리

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
date: 2023-04-02 22:00:00 +0900

# seo
# if not specified, date will be used.
# meta_modify_date: 2023-03-24 15:00:00 +0900
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

학교에서 공부하는 Ubuntu 서버를 정리하는 포스트입니다.
사용자 관리와 그룹화에 필요한 명령어들을 알아보고 파일의 소유와 허가권을 관리하는데 필요한 명령어들을 알아보고, 실습해 보겠습니다.

<!-- outline-end -->

* * *

### 개요
리눅스는 기본적으로 다중 사용자 시스템으로 제작되었습니다.   
그래서 다중 사용자를 관리하기 위해 사용자와 그룹화에 관한 명령어와 파일의 소유와 허가권과 관련 명령어도 자세히 알아보겠습니다.

### 목차

1. [사용자와 그룹](#사용자와-그룹)
2. [파일 허가권](#파일-허가권-및-소유권에-대한-실습)
3. [링크](#링크에-관해)
4. [프로세스](#프로세스)

### 사용자와 그룹
#### 관련 개념 및 관련 파일 위치
**사용자 파일**
- 텍스트 에디터에서 `/etc/passwd`의 파일을 열면 각 행은 '사용자 이름: 비밀번호:사용자 ID:사용자 소속 그룹 ID: 추가 정보: 홈 디렉터리: 기본 셸'을 의미합니다.   
- 실제 예시: `ubuntu:x:1000:1000:,,,:/home/tony /bin/bash`
    - 사용자 이름은 tony, 비밀번호는 x(/etc/shadow 파일에 비밀번호가 지정되어 있음), 사용자 ID와 그룹 ID는 1000이고, 추가 정보는 '이름, 방 번호, 직장 전화번호, 집 전화번호, 기타'가 있지만 이름을 제외한 정보는 생략되었습니다. 이후에는 홈 디렉터리가 /home/ubuntu이고, 로그인 셸은 /bin/bash임을 알 수 있습니다.
- root 사용자는 사용자 ID와 소속 그룹 ID가 모두 0으로 설정되어 있음을 확인할 수 있습니다.

**그룹 파일**
- 텍스트 에디터로 `/etc/group`파일을 열면 각 행은 '그룹 이름:비밀번호:그룹 ID:보조 그룹 사용자'를 의미합니다.
    - 보조 그룹 사용자는 이 그룹을 보조로 사용하는 사용자의 목록이고, 여러 명이면 ','로 구분합니다.
- 실제 예시: `tony:x:1000:`
    - 그룹 이름은 tony이고, 비밀번호도 있고, 그룹 ID는 1000이며, 보조 그룹 사용자는 없음을 알 수 있습니다.

#### 관련 명령어
**adduser**  
새로운 사용자를 추가하는 명령어입니다. 해당 명령어를 실행하면 /etc/passwd, /etc/shadow, /etc/group 파일에 새로운 행이 추가됩니다.   

| 명령어 | 설명 |
| :---: | :---: |
| adduser testuser1 | testuser1 사용자 생성 |
| adduser --uid 1111 testuser2 | testuser2 |사용자를 생성하고 사용자 ID를 1111로 지정
| adduser --gid 1000 testuser3 | testuser3 사용자를 생성하고, 그룹 ID가 1000인 그룹에 포함 |
| adduser --home /newhome testuser4 | testuser4 사용자를 생성하고 홈 디렉터리를 /newhome으로 지정 |
| adduser --shell /bin/csh testuser5 | testuser5 사용자를 생성하고 기본 셸을 /bin/csh로 지정 |

**passwd**   
사용자의 비밀번호를 변경하는 명렁어 입니다.  

| 명령어 | 설명 |
| :---: | :---: |
| passwd testuser1 | testuser1 사용자의 비밀번호를 설정 또는 변경 |

**usermod**  
사용자의 속성을 변경하는 명령어입니다.

| 명령어 | 설명 |
| :---: | :---: |
| usermod --shell /bin/csh testuser1 | testuser1 사용자의 기본 셸을 /bin/chs로 변경 |
| usermod --groups tony testuser1 | testuser1 사용자의 보조 그룹에 test 그룹 추가 |

**userdel**   
사용자를 삭제하는 명렁어입니다.

| 명령어 | 설명 |
| :---: | :---: |
| userdel testuser2 | testuser2 사용자 삭제 |
| userdel -r testuser3 | testuser3 사용자 삭제 및 홈 디렉터리 삭제 |

**chage**  
사용자의 비밀번호를 주기적으로 변경하도록 설정하는 명령어입니다.

| 명령어 | 설명 |
| :---: | :---: |
| change -l testuser1 | testuser1 사용자에 설정된 내용 확인 |
| change -m testuser1 | testuser1 사용자에 설정한 비밀번호를 사용해야 하는 최소 일자(변경 후 최소 2일은 사용해야 함) |
| change -M 30 testuser1 | testuser1 사용자에 설정한 비밀번호를 사용할 수 있는 최대 일자(변경 후 30일 뒤에는 사용 불가능) |
| change -E 2025/12/12 testuser1 | testuser1 사용자에 설정한 비밀번호 만료일 설정(2025년 12월 12일까지 사용 가능) |
| change -W 10 testuser1 | testuser1 사용자에 설정한 비밀번호 만료 전의 경고 기간(암호 만료일 10일 전부터 경고 메시지 출력). 디폴트 값은 7일 |

**groups**  
사용자가 소속된 그룹을 보여주는 명령어입니다.

| 명령어 | 설명 |
| :---: | :---: |
| groups | 현재 사용자가 소속된 그룹을 출력 |
| groups testuser1 | testuser1 사용자가 소속된 그룹 출력 |

**groupaadd**   
새로운 그룹을 생성하는 명령어입니다.

| 명령어 | 설명 |
| :---: | :---: |
| groupadd newgroup1 | newgroup1 그룹 생성 |
| groupsadd --gid 2222 testuser1 | newgroup2 그룹을 생성하고 그룹 ID를 2222로 지정 |


**groupmod**  
그룹의 속성을 변경하는 명령어입니다.  

| 명령어 | 설명 |
| :---: | :---: |
| groupmod --new-name mygroup1 newgroup1 | newgroup1 그룹의 이름을 mygroup1로 변경 |

**groupdel**   
그룹을 삭제하는 명령어입니다.

| 명령어 | 설명 |
| :---: | :---: |
| groupdel newgroup2 | newgroup2 그룹 삭제(newgroup2 그룹을 주 그룹으로 지정한 사용자가 없어야 함) |

**gpasswd**   
그룹의 비밀번호를 설정하거나 그룹 관리를 수행하는 명령어입니다.

| 명령어 | 설명 |
| :---: | :---: |
| gpasswd mygroup1 | mygroup1 그룹의 비밀번호 변경 |
| gpasswd -A testuser1 mygroup1 | testuser1 사용자를 mygroup1 그룹 관리자로 지정 |
| gpasswd -a testuser4 mygroup1 | testuser4 사용자를 mygroup1 그룹에 추가 |
| gpasswd -d testuser4 mygroup1 | testuser4 사용자를 mygroup1 그룹에서 제거 |


#### 실습
1. `adduser testu1` 명령을 입력하여 testu1 사용자를 생성합니다.
    - 비밀번호를 12341234로 설정하고, 나머지는 Enter를 눌러 생략합니다. 마지막으로 정보가 올바르냐는 물음에 올바르면 y를 누르면 사용자가 만들어집니다.
2. `tail /etc/passwd`로 testu1 사용자가 만들어졌는지 확인합니다.
    - `testu1:x:1001:1001:,,,:/home/testu1:/bin/bash`로 유저, 그룹 ID가 1001인 testu1이 정상적으로 만들어진 것을 확인할 수 있습니다.
3. `tail /etc/group`로 그룹을 확인합니다.
    - `testu1:x:1001:`로 그룹 이름이 testu1이고, 그룹 ID가 1001인 그룹이 생성되었음을 알 수 있습니다.
    - adduser 명령 실행시 별도로 그룹을 지정하지 않으면 자동으로 사용자 이름과 동일한 그룹이 생성되고, 새로운 사용자는 생성된 그룹에 자동으로 포함됩니다.
    - 이럼 많은 사용자를 관리할 때 사용자와 그룹의 이름이 같아 관리가 불편하기 때문에 그룹을 먼저 만든 후 사용자를 그 그룹에 넣는 것이 좋습니다.
4. `userdel -r testu1`으로 testu1 사용자를 삭제하고, `groupadd testGroup` 명령으로 testGroup 그룹을 만든 후, `tail -5 /etc/group` 명령으로 그룹이 잘 만들어졌는지 확인합니다.
    - `testGroup:x:1001:`으로 정상적으로 만들어 진 것을 확인할 수 있습니다.
5. `adduser --gid 1001 testu1`, `adduser --gid 1001 testu2`로 testGroup에 testu1과 testu2 사용자를 생성합니다.
    - `tail -5 /etc/passwd`로 유저가 정상적으로 저장되었는지 확인합니다.
    - `tail -5 /etc/shadow`로 각 유저의 비밀번호가 salt가 적용된 암호화 방식으로 암호화되 저장된 것을 보면 비밀번호가 설정되어 있음을 확인할 수 있습니다.

**X윈도우 환경에서 사용자 관리하기**
- 설정 -> 사용자에서 관리 가능

### 파일 권한
**파일 상세 정보 설명**
- `touch testtxt.txt`로 빈 파일을 만들고, `ls -l testtxt.txt`로 파일의 상세 정보를 확인해 보면 아래와 같이 나옵니다.
- `-rw-r--r-- 1 root root 0 4월 3 19:33 testtxt.txt`
    - -: 파일 유형
    - rw-r--r--: 파일 허가권
    - 1: 하드링크 수
    - root root: 파일 소유자, 파일 소유 그룹
    - 0: 파일 크기(byte)
    - 4월 3 19:33: 마지막 변경 날짜 및 시간
    - testtxt.txt: 파일 이름

**파일 허가권**
- 'rw-', 'r--', 'r--'과 같이 3개씩 끊어서 첫 번째가 소유자, 두 번째 그룹, 세 번째 그 외 사용자의 권한을 뜻합니다.
- r은 read, w는 write, x는 execute의 약자입니다. 즉 읽고, 쓰고, 실행하는 권한을 표현합니다.
- 또한 각 권한은 2진수로 표현되기 때문에 rwx 순으로 111이 되어 r은 4, w는 2, x는 1로 표현됩니다. 즉 rw-는 6, r--는 4, r-x는 5와 같이 표현할 수 있습니다.
- 디렉터리를 해당 디렉터리로 이동하려면 실행 권한이 반드시 있어야 해서, 일반적으로 디렉터리에는 소유자, 그룹, 그 외 사용자의 실행 권한이 설정됩니다.

**chmod**
- 파일 허가권을 변경하는 명령어입니다.

| 명령어 | 설명 |
| :---: | :---: |
| chmod 777 testtxt.txt | testtxt.txt에 대해 모든 사용자에게 읽고, 쓰고, 실행할 수 있는 기회를 주라는 의미(8진수 모드)|
| chmod u+x testtxt.txt | testtxt.txt에 대해 소유자에게 실행 권한을 허가하라는 의미 (u(소유주), g(그룹), o(그 외), a(모두)), (+(권한 추가), -(권한 삭제), =(권한 부여)), (r, w, x)|
| chown tony testtxt.txt | testtxt.txt 파일의 소유자를 tony로 바꾸라는 의미 |
| chown tony.tony testtxt.txt | 소유자와 그룹 모두 tony로 변경하라는 의미 |
| chgrp tony testtxt.txt | 그룹만 tony 그룹으로 바꾸라는 의미 |

#### 파일 허가권 및 소유권에 대한 실습
1. 테스트에 사용할 txt 파일을 `vi sample` 명령을 통해 생성하고, 간단히 작성한 후 저장합니다.
2. ls -l sample 명령을 입력해 파일 속성을 확인합니다.
    - `-rw-r--r-- 1 root root 49 4월 2 20:11 sample` 허가권은 'rw-r--r--'고, 소유자와 그룹은 root인 sample 파일이 잘 생성되었습니다.
3. `./sample`을 통해 파일을 실행하면 허가 거부 메시지가 나타납니다.
    - 현재 사용자는 `whoami` 명령어를 통해 알 수 있습니다.
4. `chmod 755 sample`을 통해 허가권을 'rwxr-xr-x'로 변경시고, ./sample 명령을 입력하면 파일이 정상적으로 실행되는 것을 확인할 수 있습니다.
5. `chown tony sample`과 `chgrp tony sample`로 소유권과 그룹을 따로 변경하거나 `chown tony.tony sample`을 통해 한 번에 변경시킬 수 있습니다.
6. `su - tony`로 tony 사용자로 접속한 후, pwd로 현재 디렉터리를 확인합니다. `ls -l /root/sample`을 해보면 해당 파일에 접근할 수 없다고 허가 거부 에러가 뜹니다.
    - `ls -ld /root`를 통해 root 디렉터리의 속성을 확인해 보면 `drwx------ 4 root root 4096  4월  3 20:46 /root`로 이 외 사용자의 허가권이 ---가 되어 읽기, 쓰기, 실행이 불가능해 tony 사용자의 /root 디렉터리 접근이 거부되었음을 확인할 수 있습니다.
7. `exit` -> `mv sample ~tony` -> `su - tony` -> `ls -l sample chmod 777 sample` -> `ls -l sample`
    - 다시 루트로 돌아가서 sample을 tony 사용자의 홈 디렉터리로 옮기고, 다시 tony 사용자 바꿔 sample 파일의 권한을 rwxrwxrwx로 변환한 뒤 파일의 상세 정보를 확인합니다.
    - `-rwxrwxrwx 1 tony tony 49  4월  2 20:11 sample`임을 확인할 수 있습니다.
8. `chown root.root sample` 명령을 입력하면 명령을 허용하지 않음 이라는 메시지가 뜹니다.

### 링크에 관해
#### 링크의 종류
파일의 링크는 하드 링크(Hard Link)와 심벌릭 링크(소프트 링크(Symbolic Link, Soft Link))로 구분됩니다.  
하드 링크는 `ln 원본파일 링크파일명`으로 기본 생성하면 만들어 지지만 심벌릭 링크는 `ln -s 원본파일 링크파일`로 옵션을 붙여야 합니다.   
\n
하드 링크 파일 -> inode 1 -> 원본 파일 데이터(데이터 블록)  
심벌릭 링크 파일 -> inode2 -> 원본 파일 포인터 -> 디렉터리에 있는 원본 파일 -> inode1 -> 원본 파일 데이터(데이터 블록)

* inode: 커널이 부여하는 파일과 관련된 모든 속성 정보를 가지고 있습니다.
* 심벌릭 링크는 windows의 바로가기와 같이 원본 파일의 경로를 저장합니다.

#### 실습
1. `cd` -> `mkdir linkdir` -> `cd linkdir` -> `vi originalfile` -> `cat originalfile`
    - 디렉터리를 홈 디렉터리로 놓고, link파일을 넣을 linkdir 폴더를 만듭니다. 해당 폴더에 링크를 진행할 원볼 파일인 originalfile을 만들고 잘 만들어 졌는지 확인합니다.
2. `ln originalfile hardlink` -> `ln -s originalfile softlink` -> `ls -il` -> `cat hardlink` -> `cat softlink`
    - 하드 링크와 소프트 링크를 생성하고, 자세한 파일 설명 맨 앞에 inode 번호를 출력합니다. 그럼 하드 링크와 오리지널 파일은 inode 번호가 같지만 softlink는 inode가 다르다는 것을 알 수 있습니다. 하지만 각 내용을 확인하면 모두 오리지널 파일의 내용이 그대로 나오는 것을 확인할 수 있습니다.
3. `mv originalfile ../ ` -> `ls -il` -> `cat hardlink` -> `cat softlink` -> `mv ../originalfile .`
    - 원본 파일을 다른 곳으로 이동시키고 하드와 심벌릭 링크 파일을 확인하면 하드 링크는 정상 출력 되지만 소프트 링크는 파일을 찾지 못하는 것을 확인하실 수 있습니다.
    - 원본 파일을 현재 디렉터리에 다시 가져오면 심벌릭 링크가 원상태로 복구됩니다.

**처음 ls -il 결과**
```
958117 -rw-r--r-- 2 root root 38  4월  3 21:48 hardlink
958117 -rw-r--r-- 2 root root 38  4월  3 21:48 originalfile
956035 lrwxrwxrwx 1 root root 12  4월  3 22:15 softlink -> originalfile
```

### 프로세스
**프로세스 정의**   
하드디스크에 저장된 실행 코드(프로그램)가 메모리에 로딩되어 활성화 한 것을 프로세스라 합니다.  
즉 명령어창이 3개 떠있다면 컴퓨터에는 명령어창이라는 프로그램 1개가 돌아가고, 프로세스로는 3개의 프로세스가 동작한다고 할 수 있습니다.   
\n
**포그라운드 프로세스**   
화면에 나타나서 사용자와 상호 작용을 하는 프로세스  
\n
**백그라운드 프로세스**
화면에 나타나지 않은 채 뒤에서 실행되는 프로세스  
\n
* 리눅스에서 프로세스 관리는 pcb(Process Control Block)에서 진행합니다.

#### 프로세스 관련 용어
- 프로세스 번호: 각 프로세스에 할당된 고우 번호로서 메모리에 로딩되어 활성화된 프로세스를 구분하기 위해 사용됩니다.
- 작업 번호: 현재 실행 중인 백그라운드 프로세스의 순차 번호입니다.
- 부모 프로세스와 자식 프로세스: 모든 프로세스는 독립적으로 실행되지 않고, 부모 프로세스에 종속되어 실행됩니다. 부모 프로세스를 종료하면 부모 프로세스에 종속된 자식 프로세스도 종료되는 것입니다.  
- 데몬: 서버 유형과 밀접한 관련이 있어 서비스=데몬=서버 프로세스 라고 이해해도 무방합니다. 눈에 보이진 않지만, 현재 시스템에서 동작 중인 프로세스를 이야기 하는 것으로, 백그라운드 프로세스의 일종입니다.
- 서비스: 평상시에도 늘 작동하는 서버 프로세스
    - 시스템과 별도로 구동되는 프로세스로 웹, DB, FTP 서버 등이 있습니다.
    - 주로 `systemctl start/stop/restart 서비스명` 명령으로 실행 및 종료합니다.
    - /lib/systemd/system 디렉터리에 있는 파일은 대부분 위 명령어로 실행 가능합니다.
    - 명령어는 `cd /lib/systemd/system` -> `ls *.service`로 확인하실 수 있습니다.
- 소켓: 필요할 때만 작동하는 서버 프로세스
    - 외부에서 요청하는 경우에만 systemd가 구동하고, 요청이 끝나면 소켓은 종료됩니다.
    - 소켓과 관련된 스크립트 파일은 서비스의 디렉터리와 같으며 확인은 `ls *.socket`으로 할 수 있습니다.

#### 프로세스 명령어
- `ps`: 현재 프로세스의 상태를 확인하는 명령어입니다. 프로세스 번호와 상태를 확인할 때는 `ps -ef | grep 프로세스명` 명령을 주로 사용합니다.
    - `-e` 모든 사용자가 실행하는 프로세스를 전체 다 보여달라는 옵션, `-f` fullform으로 다양한 정보를 보여주는 옵션
- `kill`: 프로세스를 강제로 종료하는 명령어
    - `-9`: 프로세스를 무조건 종료하는 옵션
- `pstree`: 부모 프로세스와 자식 프로세스의 관계를 트리 형태로 보여주는 명령어입니다.

#### 프로세스 실습
**무한 루프 도는 프로세스 중지시키기**
1. `yes > /dev/null`로 무한 루프를 도는 단순한 프로세스를 생성합니다.
    - `yes`: 화면에 yes를 계속 띄우라는 명령어입니다.
    - `/dev/null`: 휴지통과 같은 역할을 하는 디렉터리입니다.
2. 다른 프롬프트를 띄우고, `ps -ef | grep yes`로 방금 만든 프로세스의 번호를 확인합니다.
    - `root 3585 3456 97 22:51 pts/4    00:00:29 yes`
    - 위 명령어의 3585는 해당 프로세스의 pid이고, 3456은 부모 프로세스의 pid입니다.
3. `kill -9 3585`로 프로세스를 종료하고, 다시 원래 터미널을 확인하면 죽었음이란 글자가 뜹니다.
    - kill 명령어로 프로세스를 종료하면 다른 터미널에서 실행 중인 것도 자동으로 종료된 것을 확인할 수 있습니다.
    - 만약 작동 중인 포그라운드 프로세스만 종료하기 위해선 프로세스가 실행되고 있는 터미널에서 ctrl+c를 눌러야 합니다.

**프로세스 상황 바꾸기**
1. `yes > /dev/null`을 통해 포그라운드 프로세스를 생성합니다.
2. ctrl+z를 눌러 프로세스를 일시 중지시키고, `bg` 명령어를 입력하면 중지된 프로세스를 백그라운드 프로세스로 계속 실행합니다.
3. `jobs` 명령어를 입력하면 현재 실행 중인 백그라운드 프로세스를 확인할 수 있습니다. 출력 내용 중 맨 앞에 나오는 것이 작업 번호인데 `fg 작업번호`를 입력하면 다시 포그라운드 프로세스로 만들 수 있습니다.
    - `[1]+  실행중 yes > /dev/null &`
    - &는 프로세스가 백그라운드에서 동작하고 있음을 알리는 용도입니다.
4. ctrl+c를 눌러 프로세스를 종료합니다.

**명령 실행 시 처음부터 백그라운드로 실행되도록 설정하기**
1. `gedit` 명령을 입력하면 gedit가 실행되고 터미널을 더 이상 사용할 수 없습니다.
2. gedit을 종료한 후 `gedit &` 명령을 입력하면 터미널이 계속 사용되 gedit이 백그라운드 프로세스로 실행되는 것을 확인할 수 있습니다.

### 참고
#### /etc/shadow에서 나오는 정보의 설명
`tony:$6$0...:19449:0:99999:7: : :` 총 9개의 필드가 있습니다.  
1. 사용자명
2. 암호화된 패스워드
    - 패스워드 앞에 !가 있으면 잠긴 상태입니다.
3. 패스워드 최종 수정일입니다.
    - 1970-01-01로부터의 일수입니다.
4. 패스워드 최소 유지기간
    - 마지막으로 변경할 날 이후로 변경할 수 없는 일수
5. 패스워드 최대 유지기간
    - 마지막으로 변경한 날 이후로 사용 가능한 일수
6. 패스워드 만료 경고 기간
    - 패스워드 만료 전 경고 메세지가 나오는 일수
7. 계정 잠김으로 부터 남은 일수
    - 패스워드 만료 후 계정이 잠기는 일수
8. 계정 만료 일자
9. 예약 필드

#### UID
보통 UID 100번 이하는 시스템과 관련된 사용자가 들어가 있고, 보통 사용자의 경우 자동으로 1000번 이후로 설정합니다.

#### /var
실행과 관련되서 임시로 보관되야 하는 가변 파일들이 보관된 파일입니다.

### 참고 자료
1. [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)
2. [사용자들의 기본 정보, 비밀번호가 저장되어 있는 파일 - 시스코킹](https://ciscoking.tistory.com/37)