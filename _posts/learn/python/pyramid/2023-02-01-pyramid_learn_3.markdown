---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: pythonlearn
title: Python pyramid 간단 학습하기 - 실제 페이지 구축

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
date: 2023-02-01 22:00:00 +0900

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
해당 코드는 [출처 3번](#출처)의 영상을 보고 제작했습니다.

--------------------------------------------------------

### 코드 소개
#### 페이지 코드
**home.jinja2**{:data-align="center"}
```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <title> {{ greet }}, {{ name }} </title>                <!-- 1 -->
    <link rel="stylesheet" href="../static/style.css" />
</head>
<body>
<h1> {{ greet }}, {{ name }} </h1>                          <!-- 1 -->
<img src="../static/tut.jpg" alt="image" style="width: 170px;">
</body>
</html>
```
1. 백앤드 코드에서 greet와 name 변수를 받아 title과 페이지에 출력합니다.

**style.css**{:data-align="center"}
```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    background-color: lightgoldenrodyellow;
    color: red;
    font-family: sans-serif;
}
```
해당 페이지의 간단한 스타일을 정의한 css 코드입니다.

#### 서버 코드
**app3.py**{:data-align="center"}
```python
from wsgiref.simple_server import make_server
from pyramid.config import Configurator
from pyramid.view import view_config


@view_config(                           // 1
    route_name='hello',
    renderer="/templates/home.jinja2"
)
def home(request):                      // 2
    return {"greet": 'welcome', 'name': 'Tut Ankh Amun'}


if __name__ == '__main__':
    HOST = '0.0.0.0'
    PORT = 8080

    with Configurator() as config:
        config.include('pyramid_jinja2')        // 3
        config.include('pyramid_debugtoolbar')  // 4
        config.add_static_view(name='static', path='static')    // 5
        config.add_route('hello', '/')
        config.scan()
        app = config.make_wsgi_app()
    server = make_server(HOST, PORT, app)
    print(f'Starting server at {HOST}:{PORT}')
    server.serve_forever()
```
1. 라우터 이름을 hello로 하고, 렌더할 페이지의 경로를 지정해 줍니다.
2. 페이지에 보낼 변수를 설정합니다.
3. jinja2를 사용하기 위해 pyramid_jinja2를 include합니다.
4.  request되는 값과 여러 정보들을 확인할 수 있도록 하는 toolbar를 생성하기 위해 include합니다.
5. static_view에 추가해 자바 스크립트 및 CSS 파일과 같은 자원들의 경로를 지정해 정적 자원을 제공하도록 합니다.
실행이 되 localhost:8080에 접속하면 설정한 변수들에 맞는 내용들이 출력되고, 사진 또한 정상 출력 됩니다.   
그리고 사용자 기준 우측 끝 중앙을 확인하면 이상한 UI가 보이는데 그것이 toolbar입니다. 해당 UI를 클릭해 들어가면 request하는 경로와 해당 요청의 자세한 정보가 나옵니다.

### 참고
#### pyramid_jinja2, pyramid_debugtoolbar 설치 방법
1. Pycharm에서 Terminal을 엽니다.
2. `cd venv`로 사용중인 가상 환경에 접속합니다.
3. `pip install pyramid_jinja2`과 `pip install pyramid_debugtoolbar`를 실행해 라이브러리를 다운로드 받습니다.

### 출처
1. [Pyramid 학습에 도움을 준 사이트1](https://trypyramid.com/)
2. [Pyramid 학습에 도움을 준 사이트2](https://docs.pylonsproject.org/projects/pyramid/en/2.0-branch/index.html)
3. [Pyramid 학습에 도움을 준 영상](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=video&cd=&cad=rja&uact=8&ved=2ahUKEwjs5J6e6_D8AhXmf94KHRV0DWIQtwJ6BAgHEAI&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DwgMj6ZsCiBk&usg=AOvVaw1olAQEo61o8mH969UZQUI6)