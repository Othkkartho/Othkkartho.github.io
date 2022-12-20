---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: virtual
title: 스트림 방식의 보드게임 제작하기

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: 네트워크 공부
# multiple tag entries are possible
tags: [네트워크, 학교 공부]
# thumbnail image for post
img: ":/virtual_community_site/post_virtual_pic1.jpg"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-12-21 10:00:00 +0900

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

대학교 네트워크보안 수업때 과제로 제출했던 스트림 방식과 버퍼-채널 방식의 보드 게임 제작을 진행하였는데 해당 포스트에서는 스트림 방식의 게임 제작 과정과 코드, 결과, 추가하고 싶은 기능들을 설명드리겠습니다.

<!-- outline-end -->

### 게임 제작할때 고려한 것

1. 주사위는 6면체 주사위 1개로 진행합니다
2. 게임 서버는 클라이언트의 이름을 받아 등록하고, 게임을 진행해야 합니다
    - 기본적으론 여러 클라이언트가 게임을 진행하는 것을 기본으로 하지만 해당 제작이 어려울 경우 우선 2명의 게임 진행을 기본으로 합니다
3. 게임의 보드에는 4개의 이벤트가 존재합니다
    - Go는 기본적인 이벤트로 나온 주사위의 값만큼 이동합니다
    - Jump는 나온 주사위의 값만큼 한 번 더 이동하는 이벤트입니다.
    - Back는 나온 주사위의 값만큼 다시 뒤로 이동하는 이벤트입니다. 즉 원래 본인이 움직이기 전 위치로 이동합니다.
    - Skip은 무인도로 1턴을 휴식하는 이벤트입니다.
4. 이외에 상대 클라이언트와의 상호작용 이벤트 1개가 존재합니다
    - Catch는 상대 말을 잡았을 때 나오는 이벤트이고, 2가지 유형으로 제작할 계획입니다
    - 하나는 상대를 잡으면 상대가 처음으로 돌아가고, 그 칸의 이벤트를 진행합니다
    - 다른 하나는 상대방 말을 잡으면 상대가 처음으로 돌아가고, 그 칸의 이벤트를 무시합니다
5. 게임 보드의 경우 서버에서 그 크기를 결정합니다
6. 보드 안의 아이템의 경우 랜덤으로 그 값을 저장합니다

### 실제 코드
#### Sever
**StreamServer.java**{:data-align="center"}
```java:StreamServer.java
public class StreamServer {
    public static Vector<ClientHandler> clients = new Vector<>();

    private static void informNew(String name) throws IOException {
        for (ClientHandler handler : clients)
            handler.dos.writeUTF(name + " is just logged in");
    }

    public static void main(String[] args) {
        try {
            ServerSocket server = new ServerSocket(3005);
            int[] board = new int[25];
            ClientHandler.board(board);

            while (true) {
                System.out.println("Server is waiting");
                Socket socket = server.accept();
                System.out.println("client is connected: " + socket);

                DataInputStream dis = new DataInputStream(socket.getInputStream());
                DataOutputStream dos = new DataOutputStream(socket.getOutputStream());
                String name = dis.readUTF();
                System.out.println(name + ": Welcome to the server.");
                informNew(name);

                ClientHandler handler = new ClientHandler(socket, dis, dos, name, 0, board);
                Thread thread = new Thread(handler);
                System.out.println("Adding this client to client vector");
                clients.add(handler);
                thread.start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
라이브러리 설명은 [참고](#streamserverjava에-사용된-라이브러리-자세히-알아보기)에 따로 작성했습니다.

**ClientHandler.java**{:data-align="center"}
```java

```

#### Client
```java

```


### 실행 결과


### 참고
#### StreamServer.java에 사용된 라이브러리 자세히 알아보기
