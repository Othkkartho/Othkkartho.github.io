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

### 참고 자료
1. [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)
2. [사용자들의 기본 정보, 비밀번호가 저장되어 있는 파일 - 시스코킹](https://ciscoking.tistory.com/37)