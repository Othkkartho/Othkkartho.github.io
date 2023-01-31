---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: pythonlearn
title: Python pyramid 간단 학습하기 - pyramid란 & 설치 방법

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: Python pyramid 공부
# multiple tag entries are possible
tags: [python, Python pyramid 공부]
# thumbnail image for post
img: ":/python/pyramid/pyramid.jpg"
# disable comments on this page
#comments_disable: true

# publish date
date: 2023-01-31 22:00:00 +0900

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

Python의 웹 프레임 워크중 pyramid를 간단 학습한 내용을 정리한 포스트입니다.

<!-- outline-end -->

코드는 [깃허브](https://github.com/Othkkartho/pyramidProject)에 올려놨습니다.

--------------------------------------------------------

### pyramid 소개
해당 소개는 피라미드 소개 사이트에서 설명하는 내용을 간단하게 옮긴것입니다.   \n

피라미드는 Python의 웹 프레임 워크 중 하나로, 작고, 빠르며 현실적입니다.   
피라미드는 Simplicity(단순성), Minimalism(최소주의), Documentation(문서화), Speed(속도), Reliability(신뢰도), Openness(개방성)와 같은 설계 및 엔지니어링 원칙을 따릅니다.   \n

피라미드는 파이썬 3과 호환되고, GitHub 저장소에 커밋할 때마다 Python 버전의 Travis 및 Jenkins를 사용해 자동으로 테스트 되며, Pyramid 사용에 대한 문서를 제공해 신구 사용자에게 친숙해지기 위해 노력합니다.   \n

피라미드는 단일 파일 응용 프로그램 구축이 가능하며, 데코레이터로 응용프로그램을 구성할 수 있습니다.  
동적 웹 응용프로그램은 보고 있는 내용에 따라 변경될 수 있는 URL을 생성하며, 정적 자산의 사용을 위한 유연한 도구를 제공합니다.   
이외에도 대화형 개발(코드 변경사항을 자동으로 감지해 브라우저에 내용을 직서 적용할 수 있는 것), 강력한 성능의 디버그, 응용프로그램 확장 등의 기능이 있습니다.

### pyramid 설치
#### Pycharm을 이용한 Pyramid 설치(제 환경)
우선 제가 pyramid 연습을 한 환경은 python 3.10 버전에 IDE는 pycharm 을 사용했습니다.

**프로젝트 시작**{:data-align="center"}
![Pyramid 프로젝트 환경 만들기](:/python/pyramid/1/new_project.jpg){:data-align="center"}
위의 사진과 같이 Pycharm에서 Pyramid를 선택하게 되면 위의 새로운 venv를 만들어도 되고, 만들어져 있는 venv를 사용해도 됩니다만 저는 새로운 venv를 만들어 파이썬 환경을 분리해 만들었습니다.   \n

템플릿 언어는 Jinja2, Backend는 쓸일은 없지만 여기서 어떤 프로젝트를 할 지 모르기 때문에 SQLAlchemy를 선택해 생성했습니다.    \n

사실 상당히 강력한 IDE인 Pycharm을 사용했기 때문에 이러면 파이썬 pyramid를 사용해 프로그램을 만들 준비가 끝납니다.

#### visual studio code를 사용할 경우
1. 본인이 사용할 pyramid 프로젝트 파일을 선택하거나 만듭니다.
2. 메뉴바의 Terminal 탭에서 New Terminal을 누릅니다.
3. `pip install "pyramid"` 또는 `pip install "pyramid==2.0"` 를 입력해 pyramid 라이브러리를 다운로드 합니다.
    - 만약 pip를 업데이트 하라는 말이 나오면 `python.exe -m pip install --upgrade pip`를 입력하셔도 됩니다만 입력하시지 않아도 오류가 나온게 아니라면 이미 다운로드 된 상태일 것입니다.

#### anaconda를 활용한 jupyter notebook을 사용할 경우
1. Anaconda Navigator에서 Environments를 누르고, Create를 통해 새로운 환경을 만들어 파이썬 환경을 독립적으로 운용시킵니다.
    - 필요없다고 생각하시면 굳이 안하셔도 됩니다.
    - 가상환경은 `conda create -n {envname}` 명령어로도 만들 수 있습니다.
2. Anaconda Prompt를 열어 본인이 만든 env로 이동 후 `pip install pyramid`를 통해 pyramid를 다운로드 합니다.
    - `conda env list` 또는 `conda info --envs`를 통해 본인의 가상 환경 목록을 확인할 수 있습니다.
    - `conda activate {envname}`을 통해 본인이 사용할 가상환경을 선택할 수 있습니다.

이상으로 설치 방법에 대한 이야기는 마치고 다음 포스트에서 단일 파일로 제작한 Pyramid 응용 프로그램을 포스트하겠습니다.

### 참고
#### Jinja2 VS Chameleon
**Jinja2**는 빠르고, 표현력이 뛰어나며, 확장이 가능한 템플릿 엔진입니다.   
특별한 템플릿의 자리 표시자를 사용하면 Python과 유사한 코드를 작성할 수 있고, 이 전달된 데이터가 최종 문서에 렌더링 됩니다.   \n

**Chameleon**은 Python을 위한 HTML/XML 템플릿 엔진으로써 HTML 태그 또는 XML 형태로 구동됩니다.   
해당 템플릿 엔진은 템플릿을 Python 바이트 코드로 컴파일하고 최적화합니다.   

#### SQLAlchemy VS ZODB
**SQLAlchemy**은 Python 프로그래밍 언어용 오픈 소스 SQL 툿킷이자 객체 관계형 매퍼입니다.   \n

**ZODB**은 파이썬을 위한 객체 데이터베이스입니다. 대한 DB는 파이썬 2.7 또는 3.4 이상의 버전에서 실행됩니다.

### 출처
1. [Pyramid 학습에 도움을 준 사이트1](https://trypyramid.com/)
2. [Pyramid 학습에 도움을 준 사이트2](https://docs.pylonsproject.org/projects/pyramid/en/2.0-branch/index.html)
3. [Pyramid 학습에 도움을 준 영상](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=video&cd=&cad=rja&uact=8&ved=2ahUKEwjs5J6e6_D8AhXmf94KHRV0DWIQtwJ6BAgHEAI&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DwgMj6ZsCiBk&usg=AOvVaw1olAQEo61o8mH969UZQUI6)