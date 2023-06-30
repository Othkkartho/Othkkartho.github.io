---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(8)-디스크 관리

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
date: 2023-06-29 22:00:00 +0900

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

이번 포스트에서는 디스크와 파티션, 디스크 추가, 공간 할당하기, RAID 구축을 진행해 보도록 하겠습니다.

<!-- outline-end -->

* * *

### 목차

1. [디스크와 파티션](#디스크와-파티션)
2. [디스크 추가](#디스크-추가)
3. [사용자별 공간 할당](#사용자별-공간-할당)
4. [RAID의 개념과 RAID 레벨](#raid의-개념과-raid-레벨)
5. [RAID 구축](#raid-구축)

### 디스크와 파티션
VMware의 메인보드에는 기본적으로 SATA0~3까지 4개의 슬롯과 SCSI0~3까지 4개의 슬롯이 존재합니다.  
SCSI 같은 경우 SCSI 0:0~0:15까지 1개의 슬롯에 16개의 장치를 장착할 수 있지만 각 장치의 ':7'부분은 시스템에의해 제외되어 있어 한 슬롯당 15개의 디스크를 장착할 수 있어 15X4=60개의 창치를 장착 가능합니다.  
SATA 방식의 경우 SATA 0:0~0:29까지 한 슬롯당 30개의 장치가 장착 가능하여 30X4=120개의 장치를 장착할 수 있습니다.  \n

이때 처음 설치한 /dev/sda 디스크의 경우 SCSI 0:0에 할당되어 있으며, CD/DVD롬의 경우 SATA 0:1에 자동 할당되어 있습니다.  
혹시 확인하고 싶으신 분들은 Server 설정 파일에 들어가셔서 Hard Disk와 CD/DVD를 클릭하면 보이는 'Advanced...'을 클릭하시면 확인할 수 있고, 필요시 변경할 수 있습니다. 그리고 그곳에서 Hard Disk의 경우 SCSI 0:7, 1:7, 2:7, 3:7이 Reserved되었다는 것을 확인할 수 있습니다.

### 디스크 추가
1. Server의 Edit Virtual machine settings를 선택합니다.
2. Virtual Machine Settings 창 왼쪽 아래의 Add를 클릭하고, Hardware Type 창에서는 Hard Disk가 선택된 상태로 Next를 클릭합니다.
3. Type은 SCSI가 선택된 상태로 Next 클릭, Create a new virtual disk가 선택된 상태로 Next 클릭
4. Maximum disk size에 1입력, Store virual disk as a single file 선택 후 Next 클릭
5. Specify Disk File 창에서 그대로 두고 Finish를 눌러도 되고, Disk File 이름을 바꾸려면 바꿔서 Finish 클릭
6. 그럼 New Hard Disk(SCSI)에 1GB가 등록되어 있고, Advanced...을 누르면 SCSI 0:1에 등록되어 있는것을 확인할 수 있습니다.
7. Server를 부팅하면 VMware 화면 오른쪽 위에 디스크 2개가 장착된 것을 확인할 수 있습니다.
8. 파티션 할당을 위해 터미널을 열고 아래와 같이 입력합니다.
    - `# fdisk /dev/sdb`: SCSI 0:1 디스크를 선택
    - `Command: n`: 이곳에서 m을 누르면 어떤 명령어들을 사용할 수 있는지와 해당 명령어들의 설명들이 나옵니다. n의 경우 새로운 파티션을 추가하겠다는 명령어입니다.
    - `Select: p`: Primary 파티션을 선택
        - Primary 파티션의 경우 최소 1개에서 최대 4개까지 설정가능합니다. 이를 사용해 주 파티션의 경우 운영 체제를 부팅할 수 있으며, 파일이 직접적으로 저장됩니다.
        - Extended 파티션은 직접 사용할 수 없고, 논리 파티션으로 사용됩니다. 해당 파티션은 부팅할 수 없습니다. 확장 파티션은 파일을 직접 저정하지 않고 일반 데이터, 오디오, 사진 및 파일을 저장하는 논리 파티션을 만드는데 사용됩니다.
    - `Partition number: 1`: 파티션 1번을 선택
    - `First sector: Enter`: 시작 섹터 번호 입력(파티션 하나만 할당하므로 첫 섹터로 설정)
    - `Last sector: Enter`: 마지막 섹터 번호 입력(파티션 하나만 할당하므로 마지막 섹터로 설정)
    - `Command: p`: 설정 내용 확인
    - `Command: w`: 설정 내용 저장
9. 파일 시스템 생성하기
    - `mkfs.ext4 /dev/sdb1` 또는 `mkfs -t ext4 /dev/sdb` 명령 실행
        - [ext4](#ext4)(Extended file System 4)는 리눅스의 저널링 파일 시스템중 하나입니다.
10. `mkdir /mydata`로 마운트할 /mydata 디렉터리를 생성한 뒤 `mount /dev/sdb1 /mydata`로 마운트하고, `cp /boot/vm~ /mydata/file1` 명령을 입력해 `ls -l /mydata`로 확인하면 file1 파일이 복사된 것을 확인할 수 있습니다.
    - 이러면 file1은 /dev/sda에 있는 것이 아닌 /dev/sdb에 저장되 있고, /mydata가 /mydata에서 확인할 수 있는것입니다.
    - 실제로 `umount /dev/sdb1`으로 마운트를 해제한 뒤 `ls -l /mydata`로 확인해보면 file2가 안보이는것을 알 수 있습니다. 삭제된것은 아니기 때문에 다시 마운트하면 보입니다.
11. 자동 마운트하기
    - `vi /etc/fstab`로 파일을 열고 끝부분에 '/dev/sdb1 /mydata ext4 defaults 0 0'을 추가하고, 저장한뒤 `reboot`합니다.
        - /dev/sdb1은 파일 시스템, /mydata는 마운트 포인트, ext4는 파일 시스템 타입, defaults는 옵션, 0, 0은 dump와 pass입니다.
        - 'defaults' 옵션은 rw(읽고 쓰기 권한), suid(사용자가 각각 실행 파일 소유자의 파일 시스템 권한으로 실행 파일을 실행하고 디렉토리의 동작을 변경할 수 있음), dev(파일 시스템을 포함하는 문자 또는 블록 장치는 시스템에 Local), exec(해당 파일 시스템에서 프로그램, 스크립트 또는 실행 가능한 권한을 나타내는 다른 모든 프로그램을 실행할 수 있습니다.), auto(파일 시스템이 탐지될 때 또는 명령 마운트 시 자동으로 마운트함), nouser(파일 시스템을 마운트하려면 루트여야 합니다), and async(데이터가 드라이브에 기록될 때 즉시 커밋되지 않음, 비동기)

### 사용자별 공간 할당
- 쿼터: 각 사용자가 사용할 수 있는 파일의 용량을 제한하는 것
1. `adduser --home /mydata/linux1 linux1`, `adduser --home /mydata/linux2 linux2`로 사용자 2명 생성
2. `vi /etc/fstab`의 옵션을 'defaults'에서 'defaults,usrjquota=aquota.user,jqfmt=vfsv0'와 같이 변경
    - 저널링 파일 시스템을 제공해 잘못 사용된 내용을 복구 할 때 로그 시스템을 이용해 유저 파일 사용을 기록하다가 백업할 수 있게 함. 뒤에는 형식을 작성한 것입니다.
3. `mount --options remount /mydata`로 재부팅효과를 내게 다시 마운트합니다.
    - 이후 mount 명령으로 확인해보면 /dev/sdb1 디렉터리가 쿼터용으로 마운트된 것을 확인할 수 있습니다.
4. `apt-get -y install quota`로 쿼터 패키지 설치
5. 쿼터 DB 생성
    - `cd /mydata` 쿼터용 파일 시스템이 마운트된 디렉터리로 이동
    - `quotaoff -avug`: 쿼터 종료
    - `quotacheck -augmn`: 파일 시스템의 쿼터 관련 체크
    - `rm -f aquota.*`: 생성된 쿼터 관련 파일 삭제
    - `quotacheck -augmn`: 파일 시스템의 쿼터 관련 체크
    - `touch aquota.user aquota.group` 쿼터 관련 파일 생성
    - `chmod 600 aquota.*`: 보안을 위해 소유자 외에는 접근 금지
    - `quotacheck -augmn`: 파일 시스템의 쿼터 관련 체크
    - `quotaon -avug`: 설정된 쿼터 시작
6. `edquota -u linux1`을 입력하면 gnu의 nano 에디터가 켜집니다.
7. 해당 화면에서 사용한도를 soft 15360KB(15MB), hard는 20480KB(20MB)로 수정해 Ctrl+o와 Enter를 눌러 파일을 저장하고 Ctrl+x를 눌러 편집을 종료합니다.
    - soft의 경우 사용자가 해당 용량을 넘으면 주의를 주고, hard는 넘지 못하게 막습니다.
    - hard를 넘을 시 해당 파일이 저장안되는 것이 아니라 저장 되다가 멈줍니다.
8. `quota`를 입력하면 항당된 디스크 공간을 확인할 수 있습니다. 'grace'의 경우 soft를 초과한 내용을 해당 일수 동안 사용가능합니다. 사용자는 해당 일수안에 자신의 quota(soft)사용량을 초과한 공간을 정리해야 합니다.
9. `repquota /mydata`로 사용자별 현재 사용량을 확인할 수 있습니다.
10. `edquota -p linux1 linux2`를 입력하면 linux1 사용자의 설정을 linux2 사용자에게도 적용합니다.

### RAID의 개념과 RAID 레벨
**RAID**는 여러 개의 디스크를 하나처럼 사용할 수 있도록 하는 기술입니다.  \n
**하드웨어 RAID**
- 하드웨어 제조 업체가 여러 디스크를 연결한 장비를 만들어 공급하는 것입니다. 이는 안정적이고 제조 업체의 기술 지원을 받을 수 있지만, 가격이 비싸고, 제조 업체에 따라 조작 방법이 다릅니다.
**소프트웨어 RAID**
- 고가인 하드웨어 RAID의 대안으로 나온 것으로 운영체제 안에서 구현되어 디스크를 관리합니다. 하드웨어 방식에 비하면 신뢰성과 속도등이 낮지만 적은 비용으로 안전하게 데이터를 저장할 수 있습니다.   \n
**RAID 구성 방식**은 Linear RAID, RAID 0, RAID 1, RAID 2, RAID 3, RAID 4, RAID 5, RAID 6이 있습니다.

### RAID 구축
RAID는 Linear RAID, RAID 0, RAID 1, RAID 5를 사용해 보겠습니다. 아래는 구성 사진입니다.
![RAID 구성](:/linux/ubuntu/8/RAID_str.png){:data-align="center"}

1. Virtual Machine Settings 창에서 위 [디스크 추가](#디스크-추가)에서 했던것과 같이 1GB의 디스크를 8개 추가한 뒤 OK를 누릅니다.
2. 터미널에서 `ls -l /dev/sd*`명령을 눌러 조금 전 장착한 장치가 디렉터리에 있는지 확인합니다.
3. `fdisk /dev/sdb`~`fdisk /dev/sdj`명령 까지 아래와 같은 망볍으로 RAID용 파티션을 생성합니다.
    - 'Command: n': 새로운 파티션 분할
    - 'Select: p': Primary 파티션 선택
    - 'Partition number: 1': 파티션 1번 선택
    - 'First sector: Enter': 시작 섹터 번호
    - 'Last sector: Enter': 마지막 섹터 번호
    - 'Command: t': 파일 시스템의 유형 선택
    - 'Hex code: fd': Linux raid autodetect 유형 번호 선택(L을 입력하면 전체 유형이 출력됩니다)
    - 'Command: p': 설정 내용 확인
    - 'Command: w': 설정 내용 저장
4. 모든 디스크 파티션을 생성한 후 `ls /dev/sd*`명령으로 확인하면 아래 사진과 같이 나오게 됩니다.
5. `apt-get -y install mdadm`으로 관련 패키지를 설치합니다.
6. `fdisk -l /dev/sdb ; fdisk -l /dev/sdc`으로 파티션 상태를 확인합니다.
7. `mdadm --create /dev/md9 --level=linear --raid-devices=2 /dev/sdb1 /dev/sdc1`으로 Linear RAID를 생성하고, `mdadm --detail -scan`으로 RAID를 확인합니다.
8. `mkdir /raidLinear`로 마운트할 디렉터리를 생성하고, `mount /dev/md9 /raidLinear`로 마운트합니다.
9. `vi /etc/fstab`로 '/dev/md9 /raidLinear ext4 defaults 0 0'을 추가하고 저장합니다.
10. 구축한 Linear RAID를 확인하기 위해 `mdadm --detail /dev/md9`을 입력합니다.
11. RAID 0 구축
    - `mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdd1 /dev/sde1`
    - `mkfs.ext4 /dev/md0` /dev/md0 장치 포맷
    - `mkdir /raid0`
    - `mount /dev/md0 /raid0`
    - `vi /etc/fstab`로 '/dev/md0 /raid0 ext4 defaults 0 0'을 추가하고 저장합니다.
    - `mdadm --detail /dev/md0`
12. RAID 1 구축
    - `mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdf1 /dev/sdg1` - Continue creating array? 라는 메시지가 나타나면 y를 입력해 계속 진행합니다.
    - `mkfs.ext4 /dev/md1`
    - `mkdir /raid1`
    - `mount /dev/md1 /raid1`
    - `vi /etc/fstab`로 '/dev/md1 /raid1 ext4 defaults 0 0'을 추가하고 저장합니다.
    - `mdadm --detail /dev/md1`
13. RAID 5 구축
    - `mdadm --create /dev/md5 --level=5 --raid-devices=3 /dev/sdh1 /dev/sdi1 /dev/sdj1`
    - `mkfs.ext4 /dev/md5`
    - `mkdir /raid5`
    - `mount /dev/md5 /raid5`
    - `vi /etc/fstab`로 '/dev/md5 /raid5 ext4 defaults 0 0'을 추가하고 저장합니다.
    - `mdadm --detail /dev/md5`
14. 버그 방지 설정
    - `mdadm --detail --scan`으로 나온 ARRAY 4개를 마우스로 드래그, 마우스 오른쪽 버튼을 클릭해 복사합니다.
    - `gedit /etc/mdadm/mdadm.conf`로 설정파일을 열어 각 행의 'name=server:숫자' 부분을 삭제한 뒤 복사한 내용을 붙여넣고 저장한 후 닫습니다.
    - `update-initramfs -u`로 설정 내용을 적용합니다.
15. `reboot`로 재부팅 한 뒤 `ls -l /dev/md*`와 df 명령으로 RAID 장치를 확인합니다.

**파티션 완료 후 모습**{:data-align="center"}
![파티션 완료 후 모습](:/linux/ubuntu/8/partition.png){:data-align="center"}
**LinearRaid의 상세 정보**{:data-align="center"}
![LinearRaid의 상세 정보](:/linux/ubuntu/8/linearraid.png){:data-align="center"}
**RAID0의 상세 정보**{:data-align="center"}
![RAID0의 상세 정보](:/linux/ubuntu/8/raid0.png){:data-align="center"}
**RAID1의 상세 정보**{:data-align="center"}
![RAID1의 상세 정보](:/linux/ubuntu/8/raid1.png){:data-align="center"}
**RAID5의 상세 정보**{:data-align="center"}
![RAID5의 상세 정보](:/linux/ubuntu/8/raid5.png){:data-align="center"}
**모든 설정이 끝나고 df명령으로 잘 설정 되어있는지 확인**{:data-align="center"}
![잘 설정되어 있음](:/linux/ubuntu/8/df.png){:data-align="center"}

### 참고
#### Hard Disk Interface 정리 (IDE, SATA, SCSI, SAS)
아래에 소개되는 IDE, SATA, SCSI, SAS는 하드 디스크, CD-ROM과 같은 기억 장치를 연결하는 인터페이스로 활용됩니다.  
또한 IDE(Integrated Drive Electronics)의 경우 pre-ATA(Advanced Technology Attachment)~ATA-3까지 IDE나 EIDE와 같은 이름들로 불렸기 때문에 Parallel ATA(병렬 ATA)에 관해 간단히 알아보겠습니다. 

- **PATA**
    - SATA가 도입될 때까지 40핀의 단자가 리본 케이블로 드라이브를 연결하였으며 이를 PATA인터페이스라고 부릅니다.
    - PATA 케이블은 16비트의 데이터를 한번에 전송합니다.
    - 과거에는 PATA 리본 케이블로 40선 짜리를 이용했지만, 기존 40선과 새로운 그라운드 40선을 추가해 UDMA/66이상에서는 80선짜리 케이블을 이용합니다. 이 그라운드선은 신호선끼리의 capacitive coupling을 줄여 crosstalk 노이즈를 감소시키는 역할을 합니다. 이는 전송 속도가 빨라질수록 문제가되며, UDMA4(66MB/s)이상으로 정상 동작하려면 80선이 필요합니다.
- **SATA**
    - SATA는 Serial ATA(직렬 ATA)로 이전 ATA 표준을 계승해 PATA를 대체하기 위해 고안되었습니다.
    - 직렬 ATA 어댑터와 장치들은 이름과 같이 비교적 속도가 빠른 직렬 연결을 이용해 연결됩니다.
    - SATA는 PATA와 다르게 낮은 전압 차분 신호(LVDS)방식을 사용하기 때문에 더 빠른 전송을 가능하게 합니다.
    - SATA는 호스트와 드라이브 사이가 1:1로 연결되어 다른 장치들과 케이블이나 대역을 같이 쓰지 않으며 별도의 점퍼설정이 필요없어 저장장치는 노트북과 데스크탑과 단자가 같고 odd는 단자의 크기에서 차이가 나지만 호환이 가능합니다.
- **SCSI**
    - SCSI(Small Computer System Interface)는 컴퓨터에서 주변기기를 연결하기 위한 병렬 표준 인터페이스로 I/O Bus 접속에 필요한 기계적, 전기적인 요구 사항과 모든 주변 기기 장치를 중심으로 명령어 집합에 대한 규격을 말합니다.
- **SAS**
    - SAS(Serial Attached SCSI)는 하드 드라이브와 테이프 드라이브처럼 컴퓨터 기억 장치로 데이터를 송수신할 수 있는 점대점 직렬 프로토콜입니다. SAS는 병렬 SCSI 버스 기술을 대체합니다.

#### EXT4
2008년 10월에 안정판이 도입된 64비트 기억 공간 제한을 없애고 ext3의 성능을 향상시키며, 하위호환성이 있는 확장 버전입니다. 특징은 아래와 같습니다.  
- 대형 파일 시스템: 최대 1엑사바이트의 볼륨과 최대 16테라바이트의 파일을 지원합니다.
- Extent: Extent는 ext2, 3에서 쓰이던 전통적 블록 매핑 방식을 대체하기 위한 것으로 인접한 물리적 블록의 묶음으로, 대용량 파일 접근 성능을 향상시키고 단편화를 줄입니다.
- 하위 호환성: ext3, 2 파일 시스템을 ext4로 마운트하는 것이 가능합니다.
- 지연된 할당: allocate-on-flush라는 파일 시스템 성능 기술을 사용합니다. 이는 데이터가 디스크에 쓰여지기 전까지 블록 할당을 지연시켜 실제 파일 크기에 기반해 블록 할당을 결정할 수 있게 되었고, 이로인해 향상된 블록 할당이 가능하게 되 하나의 파일에 대한 블록이 여러 곳으로 분산되는 것을 막습니다.
- 32,000개의 하위 디렉터리 제한 없음: 이 제한이 64,000개로 늘어났으며, 'dir_nlink' 기능은 이보다 더 큰 개수도 허용합니다.
- 단점으로는 지연된 할당과 데이터 유실 가능성이 있습니다.
    - 따라서 2.6.30 이상의 커널에서는 자동으로 모든 데이터가 디스크에 기록되기 전에 발생한 시스템 충돌이나 전원 차단 시 추가적인 데이터 유실 위험를 알아차리고 이전 동작으로 되돌립니다.

#### RAID 구성
- **LINEAR RAID**: 디스크 2개 부터 구성할 수 있고, 첫 번째부터 차례로 저장됩니다. 사용량은 N입니다.
- **RAID 0**: 디스크 2개 부터 구성할 수 있고, 동시 저장을 지원하며, 속도가 가장 빠릅니다. 스트라이핑 방식을 사용합니다. 사용량은 N입니다.
- **RAID 1**: 디스크 2개 부터 구성할 수 있고, 동시 저장을 지원하며, 결함 허용을 제공합니다. 미머링 방식으로 사용량은 N/2d입니다.
- **RAID 5**: 디스크 3개 이상 사용해 구성해야 하고, 패리티 비트를 사용해 결함을 허용합니다. 사용량은 N-1입니다.
- **RAID 6**: 디스크 4개 이상 사용해야 하며, 중복 패리티 비트를 사용해 결함 허용을 RAID 5보다 더 제공합니다. 사용량은 N-2입니다.

### 참고 자료
1. [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)
2. [위키백과 - 병렬 ATA](https://ko.wikipedia.org/wiki/%EB%B3%91%EB%A0%AC_ATA)
3. [위키백과 - 직렬 ATA](https://ko.wikipedia.org/wiki/%EC%A7%81%EB%A0%AC_ATA)
4. [위키백과 - SCSI](https://ko.wikipedia.org/wiki/SCSI)
5. [위키백과 - 시리얼 부착 SCSI](https://ko.wikipedia.org/wiki/%EC%8B%9C%EB%A6%AC%EC%96%BC_%EB%B6%80%EC%B0%A9_SCSI)
6. [EaseUS - Primary vs. Extended Partition](https://www.easeus.com/knowledge-center/primary-vs-extended-partition.html)
7. [위키백과 - ext4](https://ko.wikipedia.org/wiki/Ext4)
8. [Halo Linux Services - Defaults The default options async auto dev exec nouser rw and suid are used](https://www.halolinux.us/ubuntu-8-10-reference/defaults-the-default-options-async-auto-dev-exec-nouser-rw-and-suid-are-used.html)