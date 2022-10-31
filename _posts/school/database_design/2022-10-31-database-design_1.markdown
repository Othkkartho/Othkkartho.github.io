---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: springlearn
title: 데이터베이스 설계 정리-1

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: 데이터베이스 설계
# multiple tag entries are possible
tags: [학교 공부 정리, 데이터베이스 설계]
# thumbnail image for post
img: ":/school/database_design_1280.png"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-10-31 12:00:00 +0900

# seo
# if not specified, date will be used.
#meta_modify_date: 2022-02-10 08:11:06 +0900
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

[데이터베이스의 개요](#데이터베이스란), [용어](#용어-정리), [DBMS](#데이터베이스-관리시스템---dbms), [데이터 언어](#데이터-언어)에 관한 포스트입니다.
-----------------------------------------------------------

데이터베이스론 학교 강의에 대한 정리 포스트입니다.

* * *

<!-- outline-end -->

### 데이터베이스란
> 한  조직(enterprise)의 여러 응용 시스템들이 공유(shared)하기 위해 통합(integrated)하여 저장(stored)한 운영 데이터(operational data)의 집합
이라고 자료에는 나와 있습니다.   <br>
만약 회사가 있다면 그 회사 안의 여러 조직에서 사용하는 응용 시스템들의 데이터를 한 곳에 모아 저장한 것을 데이터라고 할 수 있습니다.<br>
그래서 데이터는 공용, 통합, 저장, 운영데이터의 성질을 가지고 있습니다.   <br>

### 용어 정리
1. 개체는 일반적으로 레코드라 이야기 하고, 데이터의 가장 작은 논리적 단위인 속성들로 구성되어 있는 DB에 표현하고자 하는 정보 객체들을 말합니다.
**학생**{:data-align="center"}
| __학번__ | 이름 | 학과 |
| :------: | :---: | :---: |
| __1111__ | Tom | Computer |
| __1112__ | Tony | Electron |
{:data-align="center"}
위와 같은 학생 릴레이션이 있을 때 개체 타입은 학번, 이름, 학과입니다. 그 밑의 각각의 값들을 개체 인스턴스라고 하며, 이 인스턴스의 집합을 개체 집합이라 합니다.
2. 관계는 개체 타입들 간의 관계인 속성 관계와 각 릴레이션들의 관계인 개체 관계가 있습니다.

3. 시스템 카탈로그는 시스템 내의 모든 객체들의 명세 정보를 수록하고 있는 시스템 테이블이며, 카탈로그에 저장된 정보를 메타 데이터라 합니다.

### 데이터베이스 관리시스템 - DBMS
과거 전통적인 응용 프로그램과 파일들은 1:1의 관계를 가졌습니다. 때문에 데이터의 종속성과 중복성의 문제가 발생했습니다.   
그래서 DBMS는 이런 데이터 종속성과 중복성 문제를 해결하기 위해 만들어진 시스템입니다. 응용 프로그램과 데이터간의 중재자 역할을 하면서 모든 응용 프로그램들이 데이터베이스를 공용할 수 있게 관리해 주는 시스템입니다.   <br>
데이터베이스는 *외부 스키마*, *개념 스키마*, *내부 스키마* 로 나누고, 이 3단계들을 Mapping해 데이터의 독립성을 지키며, 각 단계에서 데이터를 사용할 수 있도록 합니다.   <br>
결국 DBMS의 궁극적 목적은 **데이터 독립성** 입니다.

### 데이터 언어
데이터 언어는 DB의 정의, 조작, 제어의 기능을 수행합니다.   
1. **데이터 정의어(DDL)**는 DB의 구조 정의와 변경을 수행하는 언어로써 논리와 물리 구조 간 Mapping을 명세합니다.
2. **데이터 조작어(DML)**는 데이터 처리 연산의 집합으로써 데이터의 검색 삽입, 삭제 변경 연산을 수행합니다. 이 언어에는 절차적 DML과 비절차적 DML이 있습니다.
3. **데이터 제어어(DCL)**는 DB관리를 위한 각종 제어 및 통제를 정의하는 언어로, 데이터 보안, 무결성, 회복, 병행 수행을 제어합니다.

### 참고
#### 데이터의 종속성과 중복성
**데이터의 종속성**은 응용 프로그램과 데이터가 상호 의존관계가 성립되 데이터 구성이나 접근 방법이 바뀌면 응용 프로그램도 변경해야 해 프로그램 개발과 데이터 관리가 복잡해지는 문제가 있습니다.   <br>
**데이터 중복성**은 유사한 데이터가 여러 데이터 파일에 중복되어 저장되는 문제를 의미합니다.   
이는 일관성 부족과 보안의 취약 경제성 결여, 무결성 악화의 문제점을 가지고 옵니다.

#### 절차적 DML과 비절차적 DML
**절차적 DML**은 Low-Level의 데이터 언어로써 데이터의 연산을 진행할 때 *어떻게(How)*와 *무엇을(What)*을 명세해 사용해야 하는 전문지식이 필요한 언어입니다.   <br>
**비절차적 DML**은 High-Level의 데이터 언어로써 데이터의 연산을 실행할 때 *무엇을(What)*만을 명세하고, *어떻게(How)*는 시스템에 위임하는 대화식 방식이며, 질의어라고도 합니다.

#### 데이터 부속어(DSL)
DSL은 호스트 프로그램속에 삽입되 사용되는 DML을 통칭합니다. 최근에는 Data 접근용 C, C++, JAVA API 등을 많이 사용합니다.

### 출처
1. 대학교 강의 자료
2. 데이터베이스 시스템(이석호/정익사/2017)