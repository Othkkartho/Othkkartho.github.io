---
layout: links
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: id_links

# publish date (used for seo)
# if not specified, site.time will be used.
#date: 2022-03-03 12:32:00 +0000

# for override items in _data/lang/[language].yml
#title: My title
#button_name: "My button"
# for override side_and_top_nav_buttons in _data/conf/main.yml
#icon: "fa fa-bath"

# seo
# if not specified, date will be used.
#meta_modify_date: 2022-03-03 12:32:00 +0000
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
#published: false


# you can always move this content to _data/content/ folder
# just create new file at _data/content/links/[language].yml and move content below.
###########################################################
#                Links Page Data
###########################################################
page_data:
  main:
    header: "링크"
    info: "공부에 사용한 책들이나 사용하는 홈페이지의 링크들을 정리해 둔 곳입니다."

  # To change order of the Categories, simply change order. (you don't need to change list order.)
  category:
    - title: "Spring"
      type: id_spring
      color: "#00FF00"
    - title: "Node.js"
      type: id_nodejs
      color: "#009000"
    - title: "Python"
      type: id_python
      color: "#FFFF00"
    - title: "Programming"
      type: id_programming
      color: "#050dfa"

  list:
    -
    # programming
    - type: id_programming
      title: "Stack OverFlow"
      url: "https://stackoverflow.com/"
      info: "Stack Overflow는 전문적이고 열정적인 프로그래머를 위한 질문과 답변 웹사이트입니다."

    # Spring
    - type: id_spring
      title: "Spring initializr"
      url: "https://start.spring.io/"
      info: "스프링을 시작할 때 스프링을 만들어주는 사이트"
    - type: id_spring
      title: "mvnrepository"
      url: "https://mvnrepository.com/"
      info: "스프링의 라이브러리를 가져오는 사이트"
    - type: id_spring
      title: "토비의 스프링 3.1"
      url: "https://github.com/AcornPublishing/toby-spring3-1"
      info: "스프링을 공부할 때 사용한 책의 깃허브"

    # Node.js
    - type: id_nodejs
      title: "Node.js 교과서"
      url: "https://github.com/ZeroCho/nodejs-book"
      info: "Node.js 공부에 사용한 책의 깃허브"

    # Python
    - type: id_python
      title: "파이썬 머신러닝 완벽 가이드"
      url: "https://github.com/wikibook/ml-definitive-guide"
      info: "파이썬 공부에 사용한 책의 깃허브"
---
