---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: Ubuntu
title: Ubuntu 서버 구축(3)-vi 에디터 자세히 알아보기

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
date: 2023-03-23 22:00:00 +0900

# seo
# if not specified, date will be used.
meta_modify_date: 2023-03-24 15:00:00 +0900
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
vi 에디터에 대해 집중적으로 다루는 포스트입니다.

<!-- outline-end -->

* * *

### 개요
이번 포스트에서는 리눅스 및 리눅스에서 사용하는 편집기에 대해 간단히 알아보고, 그 중 모든 UNIX, Linux에서 사용되는 vi(vim) 에디터의 단축키를 집중적으로 알아보겠습니다.


### 목차

0. [유닉스(리눅스) 편집기](#유닉스리눅스-편집기)
1. [vi 편집기 소개](#vi-편집기-소개)
2. [vi의 각종 명령어](#각종-명령어-설명)
    1. [vi 종료 명령어](#vi-종료-명령어)
    2. [커서의 이동](#커서의-이동)
    3. [데이터 삽입, 삭제, 변경](#데이터-삽입-삭제-변경)
    4. [저장](#저장)
    5. [검색](#검색)
    6. [범위 지정 및 문자열 변환](#범위-지정-및-문자열-변환)
    7. [복사, 붙여넣기](#복사-붙여넣기)
    8. [이외 간단한 명령어](#이외의-여러-명령어들)
    9. [환경 설정](#환경-설정)

### 유닉스(리눅스) 편집기
- 행단위 편집기(라인 편집기)
    - ed: 초기 유닉스 시스템용 텍스트 에디터로 지금은 거의 사용하지 않습니다. 줄 단위로 데이터를 취급해 커서 이동이 줄 단위로만 가능하기 때문에 사용하기 불편합니다.
    - ex: ed 편집기를 바탕으로 기존 기능을 많이 확장하고 개선한 편집기로 단독으로 사용하기 보다 vi와 연결하여 사용됩니다.
    - sed: 스트림 편집기로 지시된 명령에 따라 파일의 내용을 일괄적으로 변경하여 출력합니다. 쉘 스크립트에서 읽어들인 파일을 편집할 때 사용합니다.
- 화면 편집기
    - vi: 모든 UNIX, Linux에 존재하며, 기본 텍스트 에디터입니다. ex를 바탕으로 만든 디스플레이 지향의 강력한 편집기입니다.
    - vim: vi의 클론 중 하나로 vi improved(vi 개선)의 약어입니다. 여러 파일 동시 편집, Syntax Highlighting(소스코드에 색깔을 덧붙임)과 같은 기능을 추가로 지원합니다.
    - emacs: vi 에디터와 쌍벽을 이루는 유닉스 '텍스트 에디터'로 막강한 기능을 제공하며, 융통성이 훌륭합니다.
- 모드형 편집기: 입력 키를 명령으로 간주하는 명령 모드와 데이터로 간주하는 입력 모드로 나뉘어 있는 편집기입니다. 예로는 vi가 있습니다.
- 비모드형 편집기: 입력하는 모든 키는 데이터로 간주되고, 명령은 특수키와 일반키의 조합으로 구성됩니다. 예로 워드가 있습니다.

### vi 편집기 소개
초기 유닉스 편집기였던 ed라는 라인 편집기를 빌 조이가 훨씬 강력한 라인 편집기인 ex(Extended Editor)를 개발했습니다.   
ex는 ed보다 훨씬 이해하기 쉽고 강력했지만 빠르면서 유연한 터미널의 기능 덕분에 향상된 기능을 전면적으로 이용할 수 있는 화면 편집기의 필요성이 제기됩니다.   
결국 빌 조이는 ex를 위한 화면지향 인터페이스인 vi(Visual Editor)를 개발했습니다.   
vi는 모든 ex 명령을 지원하지만 자신만의 독특한 명령과 화면 전체를 사용하는 기능을 가지고 있으며 유닉스 시스템 단말기에서 사용되도록 만들어진 대화식 편집기로 한 화면은 기본적으로 80 문자, 26개 라인을 나타냅니다.   
\n
vi 편집기 모드

| 모드 | 전환 키 | 설명 |
| Command Mode(명령 모드) | Esc | 저장, 문자(열) 치환, 검색 등을 하기 위한 명령어의 입력 및 실행할 수 있는 모드 |
| Insert Mode(텍스트 입력 모드) | i, a, o 등 | 문자를 입력할 수 있는 모드로, vi 모드에서 전환키를 누르면 전환되는 모드 |
| ex Mode(명령어 입력 모드) | : | 명령 모드와 비슷하지만, vi 모드에서 ':'를 누른 후 확장된 명령어를 사용할 수 있는 모드 |

### 각종 명령어 설명
#### vi 종료 명령
vi 종료 명령어들은 기본적으로 ex모드에서 구동됩니다.

| 키 | 설명 |
| :---: | :---: |
| :wq | 편집된 내용을 저장하고 종료합니다(Wirte+Quit) |
| :q | 편집기를 종료합니다 |
| :q! | 편집한 내용이 있어라도 저장하지 않고 편집기를 강제 종료합니다. |
| ZZ(Shift+zz) | Commend Mode에서 구동하는 :wq와 같은 기능을 하는 명령어로 편집된 내용을 저장하고 종료합니다 |

#### 커서의 이동
다양한 곳으로 커서를 이동시키는 명령어들이 존재하고, 기본적으로 Commend Mode에서 동작합니다.   

**한 칸씩 커서 이동**

| 키 | 기능 |
| :---: | :---: |
| h | 커서를 한 칸 왼쪽으로 이동 |
| l | 커서를 한 칸 오른쪽으로 이동 |
| j | 커서를 한 칸 아래로 이동 |
| k | 커서를 한 칸 위쪽으로 이동 |

**줄간 커서 이동**

| Shift+'-' | 커서가 위치한 줄의 맨 처음으로 이동 |
| '-' | 커서를 이전 줄의 맨 처음으로 이동 |
| Shift+'+' | 커서를 다음 줄의 맨 처음으로 이동 |
| :n | ':'를 눌러 ex 모드로 진입한 다음 원하는 행위치를 입력하고, Enter를 누르면 해당 줄로 이동 |

**글자 단위로 커서 이동**

| 키 | 기능 |
| :---: | :---: |
| w | 커서를 다음 단어의 첫 글자로 이동 |
| e | 커서를 다음 단어의 끝 글자로 이동 |
| b | 커서를 이전 단어의 첫 글자로 이동 |

**화면 스크롤 명령어**

| ctrl+d | 화면 스크롤을 반 화면 아래로 |
| ctrl+u | 화면 스크롤을 반 화면 위로 |
| ctrl+f | 화면 스크롤을 한 화면 아래로 |
| ctrl+b | 화면 스크롤을 한 화면 위로 |
| ^e(ctrl+e) or (ctrl+shift+e) | 화면 스크롤을 1단개씩 올림 |
| ^y(ctrl+y) or (ctrl+shift+y) | 화면 스크롤을 1단개씩 내림 |

#### 데이터 삽입, 삭제, 변경
데이터 삽입, 삭제, 변경은 대부분 명령 모드에서 명령을 입력하여 텍스트 입력 모드로 진입하거나, 명령 모드에서 바꿀 수 있도록 하는 명령어들이 존재합니다.   

**텍스트 입력 모드 진입**

| 키 | 기능 |
| :---: | :---: |
| i | 현재 커서 앞 자리에 데이터 삽입 |
| I | 현재 줄 첫 칸 앞에 텍스트 입력 |
| a | 현재 커서 뒷 자리에 데이터 삽입 |
| A | 현재 줄 마지막 칸에 텍스트 입력 |
| o | 커서가 있는 현재 라인의 아래 라인어 삽입 |
| O | 커서가 있는 현재 라인의 위쪽 라인에 삽입 |

**데이터 삭제하기**

| 키 | 기능 |
| :---: | :---: |
| x, #x | 문자 하나를 삭제한 후 버퍼에 저장 |
| dw, #dw | 한 단어를 삭제한 후 버퍼에 저장 |
| D(shift+d) | 커서에서부터 라인 마지막까지 삭제한 후 버퍼에 저장 |
| dd, #dd | 한 줄을 삭제하고, 버퍼에 저장 |
| u | 방금 수행한 명령 취소 |
| U(shift+u) | 해당 줄의 모든 편집 명령 취소 |

- 위 표에서 #에 원하는 정수를 집어 넣으면 해당 값 만큼 명령어가 작동합니다.   

**데이터 변경**

| 키 | 기능 |
| :---: | :---: |
| r | 현재 커서 위치의 단지 한 글자만 변경 |
| R(shift+r) | 현재 커서부터 ESC 입력까지 겹쳐 써서 변경 |
| C(shift+c) or c | 커서를 기준으로 라인 끝까지 지우고 텍스트 입력 모드로 전환 |
| cc | 커서가 놓여있는 전체 줄을 지우고 테스트 입력 모드로 전환 |
| S(shift+s) | cc와 동일한 역할 |
| s | 현재 커서부터 ESC 키를 입력할 때까지의 내용을 변경 |
| cw | 커서 위치부터 현재 단어의 끝까지 한 단어 내용을 변경 |

#### 저장
기본적으로 ex 모드에서 동작하고, [종료 명령](#vi-종료-명령)에 작성했던 명령어는 중복 작성하지 않습니다.

| 키 | 기능 |
| :---: | :---: |
| :x | 저장하고 종료 |
| :w | 저장 |
| :w 파일이름 | 지정한 파일명으로 저장 |
| :n1,n2 w 파일이름 | n1 행부터 n2 행까지 내용을 파일명으로 저장 |


#### 검색
기본적으로 명령어 모드에서 동작합니다.

| 키 | 기능 |
| :---: | :---: |
| /문자열 | 현재 위치부터 파일 아래로 문자열 탐색 |
| ?문자열 | 현재 위치부터 파일 뒤쪽으로 문자열 탐색 |
| n | 문자열 탐색 중 사용, 다음 문자열 탐색 |
| N | 문자열 탐색 중 사용, 역방향으로 문자열 탐색 |

#### 범위 지정 및 문자열 변환
ex 모드에서 동작합니다.   
**범위 지정 방법**

| 범위 | 의미 |
| :---: | :---: |
| 1,$ | 첫 줄에서 마지막 줄까지(파일내 모든 줄) |
| % | 첫 줄에서 마지막 줄까지(파일내 모든 줄) |
| 1,. | 첫 줄에서 커서가 있는 현재 줄까지 |
| .,$ | 현재 줄부터 마지막 줄까지 |
| .-2 | 현재 줄에서 앞쪽으로 2번째 한 줄 |
| 10, 20 | 10번째 줄에서 20번째 줄까지

- 위치 표현시 '.'는 현재 줄, '$'는 마지막 줄을 의미하는 특수 문자입니다.

**문자열을 찾아 다른 문자열로 변경**

| 명령어 | 설명 |
| :---: | :---: |
| :s/문자열1/문자열2/ | 커서가 위치한 줄의 첫 번째 문자열1을 문자열2로 바꿈 |
| <범위>s/문자열1/문자열2/ | <범위>안의 모든 줄에 대해서 각 줄의 첫번째 문자열 1을 찾아 문자열 2로 바꿈 |
| <범위>s/문자열1/문자열2/g | <범위>안의 모든 줄에 대해서 모든 문자열 1을 문자열 2로 바꿈 |
| <범위>s/문자열1/문자열2/gc | <범위>안의 모든 줄에 대해서 각 문자열 1을 문자열 2로 치환할 때 수정할지 안 할지를 물어봄 |

**수정시 질문 옵션**

| 옵션 | 의미 |
| :---: | :---: |
| y | 선택된 단어를 바꾸고, 다음 단어로 넘어감 |
| n | 선택된 단어를 바꾸지 않고, 다음 단어로 넘어감 |
| a | 찾은 단어를 모두 바꿈 |
| q | 찾은 단어를 바꾸지 않고, 명령 모드로 전환 |
| l | 현재 선택된 단어만 바꾸고, 명령 모드로 전환 |

<!-- - ctrl+e와 ctrl+y는 된다고 나와있지만 옵션상에서는 작동이 되지 않아 22.04에서 다시 확인할 것 -->

#### 복사, 붙여넣기
| 키 | 설명 |
| :---: | :---: |
| yw | 현재 커서부터 단어의 끝까지 복사 |
| y$ | 현재 커서부터 줄의 끝까지 복사 |
| yy | 현재 커서가 있는 줄을 복사 |
| nyy | 현재 커서부터 n번째 줄까지 복사 |
| yj | 헌재 커서 위치의 줄과 아랫줄까지 복사 |
| yk | 현재 커서 위치의 줄과 윗줄을 복사 |
| P(shift+p) | 현재 커서 위치의 행 앞(커서 위)에 버퍼 내용을 붙여 넣기 |
| p | 현재 커서 위치의 행 뒤(커서 아래)에 버퍼 내용을 붙여 넣기 |

#### 이외의 여러 명령어들
**반복 실행**
1. 커서를 첫 번째 문자열1로 이동
2. cw 명령으로 첫 번째 문자열1을 문자열2로 변경
3. ESC키를 눌러 입력 모드에서 명령 모드로 전환
4. 커서를 눌러 두 번째 문자열1로 이동
5. '.'를 눌러 문자열1을 문자열2로 변경

**취소 명령**
u를 누르면 이전 명령이 취소되고, 전 상태로 되돌립니다.

**다른 문서 끼워 넣기**
1. test라는 이름으로 문서를 하나 작성
2. 기존에 작성한 문서에서 test 문서를 삽입할 위치에 커서를 두고, :r test 명령을 입력

**읽기 전용 편집**
`vi -R [FileName]` 또는 `view [FileName]`을 사용하면 읽기 전용으로 파일을 열고, 편집한 내용을 저장하려면 다른 이름으로 저장해야 합니다.

**데이터 복구**
- `vi -r`: 되살릴 수 있는 모든 파일을 보여줍니다.
- `vi -r [FileName]`: 되살릴 수 있는 파일 중 입력한 파일 이름에 맞는 파일을 보여줍니다.

**대소문자 변경**
명령 모드에서 바꾸고 싶은 문자로 커서를 이동해 '~'(shift+'`')를 누르면 됩니다.

**파일 암호화**
1. `vi -x [Filename]` 을 입력하면 암호를 입력하라는 메시지가 나타납니다.
2. 글을 입력하고, 저장 후 종료합니다.
3. 다시 해당 파일을 열면 패스워드를 입력하라는 메시지가 나타납니다.
    - 패스워드를 잘못 입력했을 경우, 암호화된 상태로 출력됩니다.

#### 환경 설정

| 명령어 | 수행 작업 |
| :---: | :---: |
| :set nu | 파일 내용의 각 줄에 줄 번호 표시 |
| :set nonu | 줄 번호 끄기 |
| :set list | 눈에 보이지 않는 특수문자 표시 |
| :set nolist | 특수문자 보기 끄기 |
| :set showmode | 현재 모드 표시 |
| :set noshowmode | 현재 모드 표시 끄기 |
| :set | set으로 설정한 모든 vi변수 출력 |
| :set all | 모든 vi 변수와 현재 값 출력 |

### 참고
#### vi 명령어를 학습하기 위한 문서 만들기
`man vi > manvi` 으로 vi 메뉴얼을 menvi라는 이름의 파일로 만듦
- man 명령어
    - 체계화된 도움말을 보여주는 명령어입니다.

### 참고 자료
1. [리눅스 실습 for Beginner](https://www.hanbit.co.kr/store/books/look.php?p_code=B7654754187)