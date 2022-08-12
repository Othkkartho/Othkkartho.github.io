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
    header: "Links"
    info: "This is a page where you can find links to the books you used in your study or to the websites you use."

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
    - title: "etc"
      type: id_etc
      color: "#D3D3D3"

  list:
    -
    # programming
    - type: id_programming
      title: "Stack OverFlow"
      url: "https://stackoverflow.com/"
      info: "Stack Overflow is a question and answer website for professional and enthusiastic programmers."

    # Spring
    - type: id_spring
      title: "Spring initializr"
      url: "https://start.spring.io/"
      info: "A site that creates a spring when you start it"
    - type: id_spring
      title: "mvnrepository"
      url: "https://mvnrepository.com/"
      info: "A site that imports spring libraries"
    - type: id_spring
      title: "Toby's Spring 3.1"
      url: "https://github.com/AcornPublishing/toby-spring3-1"
      info: "The github of the book I used to study Spring"

    # Node.js
    - type: id_nodejs
      title: "Node.js textbook"
      url: "https://github.com/ZeroCho/nodejs-book"
      info: "Github of the book I used to study Node.js"

    # Python
    - type: id_python
      title: "Python Machine Learning Complete Guide"
      url: "https://github.com/wikibook/ml-definitive-guide"
      info: "The github of the book I used to study Python"

    # etc
    - type: id_etc
      title: "pixabay"
      url: "https://pixabay.com/"
      info: "free image sites"
---
