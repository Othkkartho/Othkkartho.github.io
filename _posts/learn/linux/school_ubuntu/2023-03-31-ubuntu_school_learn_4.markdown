---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(4)-기본 명령어, 파일 관련 명령어, 네트워크 명령어, 파이프, 필터, 리디렉션 명령어 써보기

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
date: 2023-03-31 22:00:00 +0900

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
자주 사용하는 명령어와 파일 압축, 파일 위치 탐색, 네트워크 명령어, 파이프, 필터, 리디렉션, 시스템 명령어, 네트워크, 방화벽, 서비스(데몬) 설정 명령어를 정리했습니다.

<!-- outline-end -->

* * *

### 개요
기본 명령어와 해당 명령어의 주요 옵션에 대해 간단하게 알아보고, 네트워크와 네트워크 설정을 확인해 보겠습니다.  
또한 파일 압축, 파일 위치 검색 관련 명령어를 정리합니다.   
마지막으로 시스템, 네트워크, 방화벽, 서비스(데몬) 설정과 파이프, 필터, 리디렉션의 명령어에 관해 알아봅니다.   
더 알아가기 역할을 하는 참고에는 ifconfig에서 나오는 정보들의 설명, 시스템 부팅 과정, 권한, 표준 입출력 에러, 패키지 수동 설치 방법에 관해 작성했습니다.

### 목차

0. [기본 명령어](#기본-명령어)
1. [네트워크 정보 파악](#네트워크-정보-파악)
2. [파일 압축](#파일-압축)
3. [파일 위치 검색](#파일-위치-검색)
4. [여러 설정 및 기타 명령어들](#시스템-네트워크-방화벽-서비스데몬-파이프-필터-리디렉션-명령어)

### 기본 명령어

| 명령어 | 주요 옵션 | 설명 |
| :---: | :---: | :---: |
| ls |  | list의 약자, 해당 디렉터리에 있는 파일 목록을 나열합니다. |
|  | -a | a는 숨김 파일을 포함해 화면에 보여줍니다. |
|  | -l | l은 현재 디렉터리의 파일의 세부 정보를 보여줍니다. |
| cd |  | change directory의 약자, 디렉터리를 이동하는 명령어입니다. |
| pwd |  | print working directory의 약자, 현재 디렉터리의 전체 경로를 화면에 출력합니다. |
| touch |  | 원래 이미 존재하는 파일의 최종 수정 시간을 변경하지만, 주로 크기가 0인 새 파일을 생성하는 용도로 사용됩니다. |
| mkdir |  | make directory의 약자, 새로운 디렉터리를 생성합니다. 그 디렉터리는 명령을 실행한 사용자의 소유입니다. |
|  | -p | parents의 약자로 부모 디렉터리가 없다면 해당 디렉터리도 자동으로 생성합니다. |
| rmdir |  | remove directory의 약자, 디렉터리를 삭제할 때 사용합니다. 파일이 들어있는 디렉터리는 삭제할 수 없습니다. |
| cp |  | copy의 약자로 파일이나 디렉터리를 복사합니다. 복사한 파일은 복사한 사용자의 소유이며, 명령을 실행하는 사용자에게 해당 파일에 대한 읽기 권한이 있어야 합니다. |
|  | -r | recursive의 약자로 디렉터리를 복사합니다. |
| rm |  | remove의 약자로 파일이나 디렉터리를 삭제합니다. 해당 디렉터리에 대한 삭제 권한이 있어야 합니다. |
|  | -i | 삭제 시 정말 삭제할지 확인하는 메시지를 출력합니다. |
|  | -f | force의 약자로 삭제시 확인하지 않고 바로 삭제합니다. |
|  | -r | recursive 해당 디렉터리와 그 하위 디렉터리를 강제로 모두 삭제합니다. |
| mv |  | move의 약자로, 파일이나 디렉터리 이름을 변경하거나 다른 디렉터리로 이동할 때 사용합니다. |
| cat |  | concatenate의 약자, 파일의 내용을 화면에 출력합니다. 명령어 뒤에 여러 개의 파일명을 나열하면 파일을 연결하여 내용을 화면에 출력합니다. |
| head, tail | -n(숫자) | 텍스트 형식으로 작성된 파일의 앞 10행 또는 마지막 10행만 화면에 출력합니다. n에 숫자를 입력하면 해당 행까지만 화면에 출력합니다. |
| more | +n(숫자) | 텍스트 형식으로 작성된 파일을 페이지 단위로 화면에 출력합니다. 만약 옵션 n에 숫자를 입력하면 해당 파일의 해당 행부터 출력하라는 의미입니다. Space Bar는 다음 페이지로 이동, B는 앞 페이지로 이동, Q를 누르면 종료합니다. |
| less | +n(숫자) | more 명령어에서 더 확장된 기능을 하는 명령어로 more에서 사용하는 명령어에 더해 위, 아래, 양옆 방향키, PageUp, PageDown도 사용 가능합니다. 옵션도 more과 동일합니다. |
| file |  | 해당 파일이 어떤 종류인지 보여줍니다. 예를들어 text 파일을 해당 명령어로 입력하면 파일이름: UTF-8 Unicode text 가 출력됩니다. |
| clear |  | 현재 사용 중인 터미널 화면을 지워줍니다. |

### 네트워크 정보 파악
호스트 OS에서 게스트 OS에 할당된 인터넷 정보를 알기 위해선 호스트 OS의 cmd창에 ipconfig 혹은 ipconfig/all을 입력한 후 VMware virtual Ethernet Adapter for VMnet8 아래의 정보를 확인하면 됩니다.   
그럼 여러 정보들이 뜨는데 IPv4 주소가 192.168.\*\*\*.1로 되어 있을 것입니다. 그럼 IP 주소는 192.~.3~192.~.254까지 할당되고, 게이트웨이와 DNS 서버는 192.168.\*\*\*.2가 할당됨을 의미합니다.   
\n
실제 가상 머신에 할당된 네트워크 정보들을 확인해 보겠습니다.
1. 모든 가상머신을 부팅하고, 터미널에서 `ifconfig ens33` 또는 `ifconfig` 명령을 입력합니다. 그럼 여러 정보가 나오는데 inet이 해당 게스트OS의 IP주소가 됩니다.
    - ifconfig 뒤의 ens의 경우 각 OS 버전마다 다를 수 있습니다. 보통은 ens32 또는 ens33 장치가 들어가 있습니다. `ifconfig ens33` 명령이 안될경우 `ifconfig`를 통해 해당 머신에 설치된 장치가 어떤 것인지 확인하시기 바랍니다.
    - 이외에도 `ip addr` 명령으로도 IP 주소를 확인할 수 있습니다.
2. 게이트웨이 정보의 경우 `ip route`, DNS 정보는 `systemd-resolve --status ens33` 명령으로 확인이 가능합니다.
3. 네트워크 관련 작업을 실행하려면 `nm-connection-editor` 명령으로 설정할 수 있습니다.
    - 자동, 고정 IP 주소 사용 결정
    - IP 주소, 서브넷 마스크, 게이트웨이, DNS 정보 입력
    - 네트워크 카드 드라이버 설정, 네트워크 장치 설정
    - 해당 명령어는 server(X윈도우 환경)에서 사용할 수 있는 명령어입니다.

### 네트워크 관련 명령어 정리

| 명령어 | 설명 |
| :---: | :---: |
| nm-connection-editor | X윈도우 환경에서 네트워크와 관련된 대부분의 작업을 수행 |
| systemctl start/stop/restart/status networking | 네트워크 시작, 정지, 재시작, 상태 보기를 할 수 있는 명령어입니다 |
| ifconfig 장치명, ifconfig | 해당 장치의 네트워크 관련 정보를 출력하거나 OS에 있는 전체 장치의 네트워크 관련 정보를 출력합니다 |
| nslookup | DNS 서버의 작동을 테스트하는 명령어로 해당 콘솔로 넘어갑니다. 옆 커서가 >로 변경되면 정상적으로 콘솔 변경이 완료 된 것입니다 |
| ping IP 주소, URL | window의 ping 명령과 동일한 임무를 수행합니다. 컴퓨터가 네트워크상에서 응답하는지 테스트해 상대컴퓨터가 네트워크상에서 정상 작동되는지 확인할 때 주로 사용합니다 |

**네트워크 관련 파일**
- /etc/resolv.conf: DNS 서버의 정보와 호스트 이름이 들어있는 파일입니다. 임시로 사용되는 파일로써 네트워크를 재시작하면 초기화됩니다.
- /etc/hosts/: 현재 컴퓨터의 호스트 이름과 FQDN(Fully Qualified Domain Name, 전체 주소 도메인 이름)이 들어있는 파일입니다.

### Server를 고정 주소로 변경해보기
1. Server의 IP정보를 확인합니다.
2. `nm-connection-editor`를 입력해 네트워크 연결 창에서 유선 연결 -> 설정 -> IPv4 설정에 들어갑니다.
3. 방식을 수동으로 변경하고, 주소를 192.168.\*\*\*.100, 넷마스크는 255.255.255.0, 게이트웨이는 '192.168.\*\*\*.2를 입력하고, DNS 서버에는 구글 DNS 서버인 8.8.8.8을 입력합니다.
    - \*\*\*에는 확인한 본인의 컴퓨터 IP 주소를 입력해야 합니다.
4. 컴퓨터를 `reboot` 하고, ifconfig로 inet와 netmask를 확인하고, `netstat -rn`으로 게이트웨이, `systemd-resolve --status ens33` 또는 `cat /etc/resolv.conf` 로 DNS Servers를 확인해 정상적으로 바뀐 것을 확인합니다.
5. firefox 브라우저를 클릭해 아무 사이트나 들어가 봐서 정상적으로 들어가지면 고정 주소로 훌륭히 변경 된 것입니다.

### DNS 주소 잘못 입력하고, 복구하기
1. `nm-connection-editor`을 입력해 IPv4 설정에 DNS 서버를 100.100.100.100으로 입력하고 저장합니다.
2. `systemctl disable systemd-resolved`와 `systemctl stop systemd-resolved`를 입력해 관련 서비스를 중지시킵니다.
    - 이후 다시 서비스를 시작하려면 `systemctl enable systemd-resolved`, `systemctl start systemd-resolved`를 입력하면 됩니다.
3. vim 에디터로 /etc/NetworkManager/NetworkManager.conf 파일을 열어 \[main\] 아래에 dns=default를 추가하여 저장후 종료합니다.
4. DNS 서버가 설정되어 있는 resolv.conf 파일을 `rm /etc/resolv.conf`로 삭제하고, `service network-manager restart`로 network-manager 서비스를 시작하여 `cat /etc/resolv.conf`로 DNS가 100.100.100.100인 것을 확인합니다.
5. `reboot` 명령으로 재부팅 후 웹 브라우저를 실행하여 다시 사이트에 들어가보면 들어가지지 않습니다.
6. 터미널에서 nslookup을 이용해 DNS 콘솔로 이동 후server를 입력하면 현재 Server에 설정된 DNS 서버 주소가 나옵니다.
7. 이상태에서 www.google.com을 입력하면 timed out 에러가 뜹니다.
8. `server 8.8.8.8`을 입력하면 기본 DNS 서버 주소가 변경되고, 다시 www.google.com을 입력하면 해당 도메인의 IP 주소가 나오면 정상적으로 변경 된 것입니다.
9. 이제 vim 에디터로 /etc/resolv.conf 파일을 열어 nameserver를 8.8.8.8로 입력후 저장하고, 영구적으로 설정하기 위해 다시 `nm-connection-editor`를 사용해 정상적인 DNS 서버 주소인 8.8.8.8을 입력하고 저장 후 `reboot` 명령으로 재부팅합니다.

### 파일 압축

**xz**   
비교적 최신 압축 명령어이며 압축률이 뛰어납니다. 하나의 파일만 압축 가능합니다.   

| 명령어 | 옵션 | 설명 |
| :---: | :---: | :---: |
| xz 파일명 |  | 파일명.xz라는 압축 파일을 생성하고, 기존 파일을 삭제합니다 |
| unxz 파일멍.xz | -d | decompress, 파일명.xz의 압축을 풀어 파일명이라는 파일을 생성합니다 |
|  | -l | list, 파일명.xz에 포함된 파일 목록과 압축률 등을 출력합니다 |
|  | -k | keep, 압축 후 기존 파일을 삭제하지 않고 유지합니다 |

**bzip2**   
버로우즈-휠러 변환 기반의 압축 소프트웨어입니다. tar과 함께 사용하는 것이 일반적입니다. gzip이나 ZIP에 비해 대체로 압축률이 좋지만 비교적 느립니다. 하나의 파일만 압축 가능합니다.  

| 명령어 | 옵션 | 설명 |
| :---: | :---: | :---: |
| bzip2 파일명 |  | 파일명.bz2라는 압축 파일을 생성하고, 기존 파일을 삭제합니다 |
| bunzip2 파일명.bz2 | -d | decompress, 파일명.bz2의 압축을 풀어 파일명이라는 파일을 생성합니다 |
|  | -k | keep, 압축 후 기존 파일을 삭제하지 않고 유지합니다 |

- -l 명령어가 없습니다.   
\n
**gzip**   
GNU zip의 준말입니다. gzip은 여러 파일 또는 디렉터리를 하나의 파일로 압축하기 위해서 tar과 같이 사용되 .tar.gz로 압축된 파일의 경우 zip과 압축 알고리즘은 같지만 더 용량이 작습니다. 하나의 파일만 압축 가능합니다.  

| 명령어 | 옵션 | 설명 |
| :---: | :---: | :---: |
| gzip 파일명 |  | 파일명.gz라는 압축 파일을 생성하고, 기존 파일을 삭제합니다 |
| gunzip 파일명.gz | -d | decompress, 파일명.gz의 압축을 풀어 파일명이라는 파일을 생성합니다 |
|  | -l | list, 파일명.gz에 포함된 파일 목록과 압축률 등을 출력합니다 |
|  | -k | keep, 압축 후 기존 파일을 삭제하지 않고 유지합니다 |
|  | -1~9 | 압축도를 1(최소)~9(최대)까지 사용할 수 있습니다 |

**zip**   
윈도우와 호환되는 압축입니다.   

| 명령어 | 옵션 | 설명 |
| :---: | :---: | :---: |
| zip 파일명 |  | 파일명.gz라는 압축 파일을 생성하고, 기존 파일은 유지합니다 |
| unzip 파일명.zip |  | 파일명.zip의 압축을 풀어 파일명이라는 파일을 생성합니다 |
|  | -1~9 | 압축도를 1(최소)~9(최대)까지 사용할 수 있습니다 |
|  | -r | 디렉터리까지 압축이 가능합니다 |

**tar**   
tape archive의 약자로, 초기에는 테입 백업 목적으로 순차적 입출력 장치에 직접 쓰도록 개발되었으나, 현재는 배포 또는 아카이브 용도로 많은 파일을 디렉토리 구조, 파일 속성들을 보존하면서 하나의 큰 파일로 묶는데 주로 사용됩니다.  
**압축은 진행하지 않습니다.**  

| 옵션 | 설명 |
| :---: | :---: |
| c | 새로운 묶음 파일 생성 |
| x | 묶음 파일 풀기 |
| t | 묶음 파일을 풀기 전에 묶인 경로를 보여줌 |
| C(shift+c) | 지정된 디렉터리에 묶음 파일을 풀거나 묶음 파일이 있는 디렉터리에 풀기 |
| f(필수) | 묶음 파일명을 지정 |
| v | 파일을 묶거나 푸는 과정을 보여줌(생략 가능) |
| J(shift+j) | tar+xz |
| z | tar+gzip |
| j | tar+bzip2 |

| 명령어 | 설명 |
| :---: | :---: |
| tar cvf my.tar /etc/fonts | my.tar로 /fonts 디렉터리에 있는 파일들을 묶고, 압축 |
| tar cvfz my.tar.gz / etc/fonts | tar로 묶고, gzip으로 압축까지 진행 |
| tar tvf my.tar | 파일 확인 |
| tar xvf my.tar | tar 풀기 |
| tar Cxvf newdir my.tar | tar을 newdir에 풀기 |
| tar xfz my.tar.gz | gzip 압축을 풀고, tar을 풀기 |

### 파일 위치 검색
find 경로 옵션 조건 action 순서로 작성하면 됩니다.   
- 옵션: -name(이름), -user(소유자), -newer(전, 후), -perm(허가권), -size(크기)
- action: -print(기본 값), -exec(외부 명령 실행)

| 명령어 | 설명 |
| :---: | :---: |
| find /etc -name "*.conf" | /etc 디렉터리 하위에 있으며 확장명이 .conf인 파일 검색 |
| find /home -user tony | /home 디렉터리 하위에 있으며 소유자가 tony인 파일 검색 |
| find ~ -perm 644 | 현재 사용자의 홈 디렉터리 하위에 있으며 허가[권한](#권한)이 644(rw-r--r--)인 파일 검색 |
| find /usr/bin -size +10k -size -100k | /usr/bin 디렉터리 하위에 있으며 크기가 10~100KB인 파일 검색 |
| find ~ -size 0k -exec ls -l { } \; | 현재 사용자의 홈 디렉터리 하위에 있으며 크기가 0인 파일의 목록을 상세히 출력 |
| find /home -name "*.swp" -exec rm { } \; | /home 디렉터리 하위에 있으며 확장명이 *.swp인 파일 삭제 |
| which 실행파일명 | PATH에 설정된 디렉터리와 절대 경로를 포함한 위치 검색 |
| whereis 실행파일명 | 실행 파일과 소스, man 페이지 파일까지 검색 |
| locate 파일명 | updatedb 명령을 한 번 실행해야 사용 가능, 입력한 파일명의 절대 경로 검색 |

- -exec ~ \; 명령어는 그 안에 있는 명령어를 실행할 때 사용합니다.
- locate 명령이 설치되지 않았을 경우 `apt install mlocate`나 `apt install plocate`를 통해 패키지를 다운받으시면 됩니다.

### 시스템, 네트워크, 방화벽, 서비스(데몬), 파이프, 필터, 리디렉션 명령어
- `gnome-control-center`: 명령으로 설정을 열 수 있습니다.
- `nm-connection-editor`: 네트워크 설정을 열 수 있습니다.
- `gufw`, `ufw`: gufw는 GUI 기반의 우분투에서 제공하는 방화벽 기능을 설정하는 명령어이고, ufw는 텍스트 모드의 실행하면 외부에서 접속하는 모든 포트가 닫힙니다.
    - 외부에 서비스를 제공할 때 필요한 포트만 열어주는 방식으로 사용하는 것이 좋습니다.
- `kcmshell5 kcm_systemd`: 서비스(데몬)의 시작, 중지, 재시작 및 사용 여부를 설정할 때 사용합니다.
    - 실행 전에 `apt -y install kde-config-systemd kde-cli-tools`를 입력해 패키지를 설치해야 합니다.
- `ls -l /etc | less`: 파일이 너무 많아 한 페이지에 모두 담을 수 없기 때문에 한 페이지씩 나누어 보겠다는 의미입니다.
    - 파이프는 두 프로그램을 연결하는 연결 통로를 의미하며 '|'를 사용합니다.
- `ps -ef | grep bash`: 모든 프로세스 번호를 출력하고 bash라는 글자가 들어있는 프로세스만 출력합니다.
    - 필터는 필요한 것만 걸러주는 명령어로 grep, tail, wc, sort, awk, sed등이 있습니다. 주로 파이프와 같이 사용합니다.
- 리디렉션: 표준 입출력의 방향을 바꿉니다.
    - `ls -l > list.txt`: ls -l의 결과를 화면에 출력하지 않고 list.txt에 저장합니다. 만약 list.txt파일이 있으면 덮어씁니다.
    - `ls -l >> list.txt`: 위 명령과 동일하지만 list.txt파일이 있으면 덮어쓰지 않고, 기존의 내용에 이어지게 입력됩니다.
    - `sort < list.txt`: list.txt 파일을 정렬하여 화면에 출력합니다.
    - `sort <list.txt> out.txt`: list.txt 파일을 정렬한 후 out.txt 파일에 저장합니다.

### 참고
#### ifconfig 명령에서 나오는 각 정보의 설명
1. flags: 네트워크 카드의 상태를 표시합니다.
    - 저는 UP(인터페이스가 작동중입니다. 인터페이스에 주소가 구성되면 켜지는 플레그입니다), BROADCAST(브로드캐스트 패킷을 지원한다는 의미입니다), RUNNING(드라이버가 인터페이스에 자원을 할당했으며 패킷을 보내고 받을 준비가 되어 있다는 것을 알립니다), MULTICAST(멀티캐스트 패킷을 지원한다는 것을 의미하지만 인터페이스에 멀티캐스트 주소가 구성되었음을 의미하지는 않습니다)라고 나와 있었습니다.
2. mtu: Maximum Transmission Unit의 약자로 데이터 링크에서 하나의 프레임 또는 패킷에 담아 운반 가능한 최대 크기, 즉 최대 전송 단위를 의미합니다.
3. inet, inet6: 네트워크상 연결된 컴퓨터를 유일하게 구분하는 번호 체계인 IP 주소를 확인할 수 있는 정보입니다.
    - inet은 IPv4 주소로 0~255까지 4 네 부분으로 나뉘어 32비트로 구성되어 있습니다.
    - inet6은 IPv6 주소로 128비트의 주소공간을 제공합니다.
4. netmask: 넷마스크로 네트워크 규모가 결정됩니다. 실습에서는 사설 네트워크에서 C클래스를 사용하기 때문에 넷마스크가 255.255.255.0입니다.
    - 실제로는 192.168.\*\*\*.0~192.168.\*\*\*.255 총 256개의 IP 주소가 사용가능합니다. 이중 네트워크 주소인 .0, 브로드캐스트 주소인 .255, 게이트웨이로 사용할 IP 주소를 제외한 253대의 컴퓨터를 네트워크 내부에서 연결 가능합니다.
5. broadcast: 내부 네트워크의 모든 컴퓨터가 수신하는 주소입니다.
6. prefixlen: IP주소에서 서브넷 마스크로 사용될 비트 수를 보여줍니다.
7. scopeid: IPv6의 범위를 알려줍니다.
8. ether: 네트워크 인터페이스의 하드웨어 주소를 보여줍니다.
9. txqueuelen: 송신할 때 큐의 길이입니다.
10. RX packets, RX errors: 받은 패킷의 정보(패킷 개수, 크기)와 오류 개수를 보여줍니다.
11. TX packets, TX errors: 보낸 패킷의 정보(패킷 개수, 크기)와 오류 개수를 보여줍니다.

#### 시스템 부팅 과정
보통 유닉스 기반 OS는 부팅 과정 중 최초의 프로세스가 init으로, 시스템이 종료될 때까지 계속 실행하는 데몬 프로세스입니다.  
\n
하지만 현재 실습에 사용하고 있는 우분투의 경우 systemd(system demon)라는 init 시스템 대신 사용자 공간을 부트스트래핑하고 최종적으로 모든 프로세스를 관리하는 init 시스템을 사용합니다.   
\n
만약 본인 OS가 어떻게 작동되는지 동작 순서를 확인하고 싶다면 우분투 기준 `pstree`를 입력하면 확인할 수 있습니다.

#### 권한
rwx권한은 이진수로 111로 설정되기 때문에 4+2+1로 만약 rwx 권한을 모두 가지고 있으면 7, read만 있으면 4, rw만 있다면 6이 됩니다.

#### 표준 입출력 및 에러
기본적으로 리눅스는 입출력 장치도 파일의 하나로 봅니다.  
각각 파일 디스크립터를 살펴보면 표준입력(0), 표준출력(1), 표준에러(2) 로 지정되어 있습니다.  
`ls -l 2 > /dev/null`: ls -l 명령어를 실행했을 때 오류가 났다면 리눅스에서 휴지통 역할을 하는 /dev/null로 보내겠다는 의미입니다.

#### 패키지 다운로드가 안될때 수동 설치 방법(예시: plocate)
가끔 다운로드를 apt install을 하다보면 분명 [pkgs.org](https://pkgs.org/)나 [ubuntu packages](https://packages.ubuntu.com/)에서 검색이 되는데 Mirror가 이미 적용이 되어 있고, 캐시 갱신을 진행 했지만 패키지가 없다고 나올 때가 있습니다.   
이럴 경우 패키지 수동 설치를 해야할 수 있는데 간단하게 방법을 알아보겠습니다.
1. [pkgs.org](https://pkgs.org/)에 들어가 다운로드 받고 싶은 패키지 이름을 검색합니다.
2. 검색된 패키지 중 본인이 사용하고 있는 버전에 맞는 패키지를 클릭하고, Download 줄까지 페이지를 내립니다.
3. Download에서 Binary Package 란의 URL을 복사하고, 우분투에 돌아가 `wget http://archive.ubuntu.com/ubuntu/pool/main/p/plocate/plocate_1.1.15-1ubuntu2_amd64.deb`를 입력합니다.
4. 진행이 완료되 .deb 파일이 저장되면 `dpkg -i plocate_1.1.15-1ubuntu2_amd64.deb`를 입력해 패키지 수동 설치를 진행합니다.
    - 저는 의존성 문제로 실행이 안되었기 때문에 liburing2 패키지 또한 3, 4번 방법으로 먼저 수동 설치를 진행 하고, 다시 plocate에 대한 수동 설치를 진행했습니다. 이후 locate과 updatedb가 정상적으로 동작합니다.
    - .deb로 설치한 패키지 또한 일반 패키지 지우는 방식과 동일합니다. `apt remove plocate`

### 참고 자료
1. [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)
2. [init - 위키백과](https://ko.wikipedia.org/wiki/Init)
3. [systemd - 위키백과](https://ko.wikipedia.org/wiki/Systemd)
4. [ifconfig -a display flags - IBM Support Center](https://www.ibm.com/support/pages/ifconfig-display-flags)
5. [리눅스 txqueuelen 변경 - 제타위키](https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_txqueuelen_%EB%B3%80%EA%B2%BD)
6. [bzip2 - 위키백과](https://ko.wikipedia.org/wiki/Bzip2)
7. [tar (파일 포맷) - 위키백과](https://ko.wikipedia.org/wiki/Tar_(%ED%8C%8C%EC%9D%BC_%ED%8F%AC%EB%A7%B7))
8. [수동 설치 - ubuntu help](https://help.ubuntu.com/kubuntu/desktopguide/ko/manual-install.html)