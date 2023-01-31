---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: pythonlearn
title: Python pyramid 간단 학습하기 - 단일 파일로 구축

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
date: 2023-02-01 10:00:00 +0900

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
해당 포스트는 [출처 3번](#출처)의 영상을 보고 제작했습니다.

--------------------------------------------------------

### 코드 소개
#### 단일 파일 응용 프로그램 구축
**app.py**{:data-align="center"}
~~~python
from wsgiref.simple_server import make_server
from pyramid.config import Configurator
from pyramid.response import Response


def hello_world(request):           // 1
    return Response('Hello World!')


if __name__ == '__main__':          // 2
    HOST = '0.0.0.0'
    PORT = 8080

    with Configurator() as config:  // 3
        config.add_route('hello', '/')  // 4
        config.add_view(hello_world, route_name='hello')    // 5
        app = config.make_wsgi_app()            // 6
    server = make_server(HOST, PORT, app)       // 7
    print(f'Starting server at {HOST}:{PORT}')  // 8
    server.serve_forever()                      // 9
~~~
1. 함수가 호출 되었을 때 어떤 문장을 전송할 지 정하는 함수입니다.
2. 해당 모듈이 import된 경우가 아닌 인터프리터에서 직접 실행된 경우에만 코드를 실행하라는 의미에 조건문입니다.
3. import한 Configurator를 열고, 해당 라이브러리로 해야할 모든 처리가 끝난 후 자동으로 close 하게 하는 구문입니다.
4. hello라는 라우터 이름을 / 경로에 매핑해 route에 추가하기 위한 코드입니다.
    - config 변수에 action_state안 actions list info에 src 항목을 보면 hello에 / 경로를 더한다는 코드가 잘 저장되어 있습니다.
**add_route 결과**{:data-align="center"}
![hello 라우터 이름에 / 경로가 잘 매핑되어 있음](:/python/pyramid/2/add_route.jpg){:data-align="center"}
5. hello_world 함수에 hello라는 라우터 이름을 붙여 view에 더하기 위한 함수입니다.
    - add_route와 같은 위치에 하나의 리스트에 데이터가 하나 더 생기고 src에 보면 입력한 코드가 정상적으로 저장된 것을 확인할 수 있습니다.
**add_view 결과**{:data-align="center"}
![hello_world 함수 이름에 hello 라우터 이름이 잘 매핑되어 있음](:/python/pyramid/2/add_view.jpg){:data-align="center"}
6. 보류 중인 구성 문을 커밋하고, pyramid.events.ApplicationCreated 이벤트를 모든 수신기에 보냅니다.
    - 해당 이벤트는 Configurator.make_wsgi_app()가 호출도고, 인스턴스에는 WSGI 요청을 처리할 라우터의 인스턴스 속성이 있습니다.
7. host와 post를 받고, Router 클래스 변수인 app에 대한 연결을 수락하는 새 WSGI 서버를 만듭니다.
8. 서버가 정상적으로 실행되었는지 확인하기 위해 콘솔창에 해당 내용을 띄웁니다.
9. 프로세스가 중지될때까지 요청에 응답하도록 만듭니다.
정상적으로 실행되 localhost:8080에 접속해 보면 Hello World!가 정상적으로 출력 되는 것을 확인할 수 있습니다.

#### json 입력
**app2.py**{:data-align="center"}
~~~python
from wsgiref.simple_server import make_server
from pyramid.config import Configurator
from pyramid.view import view_config


@view_config(           // 1
    route_name="home",
    renderer="json"
)
def home(request):
    return {"a": 1, "b": 2}


if __name__ == '__main__':
    HOST = '0.0.0.0'
    PORT = 8080

    with Configurator() as config:
        config.add_route('home', '/')
        config.scan()                   // 2
        app = config.make_wsgi_app()
    server = make_server(HOST, PORT, app)
    print(f'Starting server at {HOST}:{PORT}')
    server.serve_forever()
~~~
1. 파일을 처음 시작하면 바로 실행되고, 뷰를 등록하는 것과 동일한 작업을 수행하도록 합니다.
    - 라우터의 이름은 home으로 지정하고, 랜더러 형식을 json 형식으로 정합니다.
2. 파이썬 패키지와 그 하위 패키지에서 객체를 스켄합니다. view_config 구성을 scan해 적용하기 위해 실행했습니다.
해당 프로그램이 실행되 localhost:8080에 접속해 보면 {"a": 1, "b": 2}가 정상적으로 출력됩니다.

### 출처
1. [Pyramid 학습에 도움을 준 사이트1](https://trypyramid.com/)
2. [Pyramid 학습에 도움을 준 사이트2](https://docs.pylonsproject.org/projects/pyramid/en/2.0-branch/index.html)
3. [Pyramid 학습에 도움을 준 영상](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=video&cd=&cad=rja&uact=8&ved=2ahUKEwjs5J6e6_D8AhXmf94KHRV0DWIQtwJ6BAgHEAI&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DwgMj6ZsCiBk&usg=AOvVaw1olAQEo61o8mH969UZQUI6)