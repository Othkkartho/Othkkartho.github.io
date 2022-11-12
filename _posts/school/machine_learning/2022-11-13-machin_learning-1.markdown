---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: machinelearn
title: 머신러닝 정리-1

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: 머신러닝
# multiple tag entries are possible
tags: [학교 공부 정리, 머신러닝]
# thumbnail image for post
img: ":/school/database_design_1280.png"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-11-13 12:00:00 +0900

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

머신러닝의 웹 크롤링에 관해 조사한 내용입니다.
-----------------------------------------------------------

그 중 Beautiful Soup과 Selenium의 설명 및 차이점 설명을 주로 작성하였습니다.

***

<!-- outline-end -->

#### 개요
 웹 크롤링은 조직적이고, 자동화된 방식으로 웹을 탐색하는 컴퓨터 프로그램인 웹 크롤러가 하는 작업을 의미합니다. 웹 크롤링은 보통 검색 엔진이나 웹을 참조해야 하는 사이트들에서 데이터의 지속적인 갱신을 통해 데이터를 최신 상태로 유지하기 위해 웹 크롤링을 진행합니다. 이외에 크롤러는 링크 체크나 HTML 코드 검증과 같은 웹 사이트 자동 유지 관리 작업과, 자동 이메일 수집과 같은 웹페이지의 특정 형태의 정보를 수집하는 데에도 사용됩니다. 크롤링에 사용하는 파이썬 라이브러리는 대표적으로 Selenium과 Beautiful Soup이 있습니다. 그럼 각 라이브러리들을 살펴보고, 그 차이점을 확인해 보겠습니다.

#### Beautiful Soup
 먼저 Beautiful Soup은 구조화된 HTML 및 XML 데이터를 수집하기 위해 명시적으로 제작된 라이브러리입니다. 이 라이브러리를 사용한다면 웹 페이지의 소스코드를 수집하고 필터링해 필요한 데이터를 얻어낼 수 있습니다. 간단한 예를 들어 보면 웹의 HTML의 ID와 Class 이름으로 각 페이지의 요소를 검색하고, 추가 처리나 재포맷을 위해 발견된 것을 출력할 수 있습니다. Beautiful Soup을 완벽하게 동작시키기 위해선 다른 파이썬 라이브러리들이 필요합니다. 만약 구문 분석을 시작하기 전 HTML 페이지 소스를 가져오려면 해당 요청 라이브러리가 필요합니다. 하지만 Beautiful Soup은 분명 실행이 매우 간단하고, 비교적 사용하기 쉽습니다.

#### Selenium
 Selenium은 자동화된 테스트를 위해 설계된 범용 웹 페이지 렌더링 도구입니다. 이 클레스는 다양한 프로그래밍 언어를 광범위하게 지원합니다. 만약 파이썬 프로그래머라면 Selenium 웹 드라이버를 가져와 다양한 로케이터를 통해 자동 스크래핑을 구현할 수 있습니다. Selenium은 JavaScript가 동적인 콘텐츠의 표시를 진행하기 전에 페이지를 먼저 로드할 때 유용한 라이브러리입니다. 여러 브라우저, 운영 체제나 스마트폰, 테블릿과 같은 PC 이외의 하드웨어 환경에서도 실행할 수 있을 만큼 여러가지 능력을 가지고 있습니다.

#### Beautiful Soup과 Selenium의 차이점
 그럼 둘의 차이점을 스크래핑 속도, 사용 편의성, 스크래핑 유연성 순으로 확인해 보겠습니다. Selenium은 JavaScript와 같은 클라이언트 측 기술이 로드 될 때까지 기다렸다가 크롤링을 진행합니다. Beautiful Soup은 페이지 소스를 크롤링 하는 것이므로 Soup의 스크래핑 속도가 더 빠릅니다. Selenium은 마우스 클릭을 시뮬레이션하고 양식을 채우는 포괄적인 웹 자동화 툴킷입니다. Beauiful Soup는 복잡하지 않은 페이지와 상호작용할 수 있습니다. 하지만 그만큼 사용하기는 더 쉽습니다. Selenium은 동적 페이지 및 콘텐츠와의 상호작용을 지원하고, 더 넓은 범위의 시나리오에서 실행될 수 있지만, Beautiful Soup는 기본적으로 정적 페이지에서 데이터를 추출하는 것으로 제한되기 때문에 피상적 웹 사이트 변경 정적 페이지는 처리가 안될 수도 있습니다. 그렇지만 이런 단순성은 프론트엔드 디자인 변경에 대해 더 탄력적으로 운용할 수 있는 이점이 있습니다. 즉 사용 편의성은 Beautiful Soup이 더 좋지만, 동적 페이지의 스크래핑 유연성은 Selenium이 더 좋다고 이야기 할 수 있습니다.    <br>
 그래서 보통 Selenium은 코드의 복잡성과, 상대적으로 느린 스크래핑 속도, 동적 페이지 및 콘텐츠와의 상호작용이 좋은 특징등 덕분에 좀 더 복잡한 프로젝트에 더 적합하고, Beautiful Soup은 빠른 스크래핑 속도, 좋은 사용 편의성, 하지만 정적 페이지에서 데이터를 추출해야 하는 장단점으로 인해 소규모 프로젝트에 더 적합합니다.   <br>
 하지만 둘 모두의 가장 큰 단점은 웹에 내재된 변동성입니다. 한 사이트에서 작동하는 스크립트가 다른 사이트에서는 작동하지 않을 수도 있고, 웹 페이지가 변경되어 실행 중이나 추후 실행에서 스크립트 오류가 발생할 수도 있어, 꾸준한 유지 관리가 필요합니다.

#### 출처
1. [셀레늄 vs. 아름다운수프 파이썬 | 전체 비교 | 퍼포스의 블레이저미터](https://www.blazemeter.com/blog/selenium-vs-beautiful-soup-python#what)
2. [웹 크롤러 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/%EC%9B%B9_%ED%81%AC%EB%A1%A4%EB%9F%AC)