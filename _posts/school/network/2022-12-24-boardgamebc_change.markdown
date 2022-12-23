---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: network
title: 버퍼/채널 방식으로 보드게임 만들기(한글 변경)

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Othkkartho's Workshop
# multiple category is not supported
category: 네트워크
# multiple tag entries are possible
tags: [네트워크, 학교 공부]
# thumbnail image for post
img: ":/virtual_community_site/post_virtual_pic1.jpg"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-12-24 20:00:00 +0900

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

대학교 네트워크보안 수업때 과제로 제출했던 스트림 방식과 버퍼-채널 방식의 보드 게임 제작을 진행하였는데 해당 포스트에서는 채널/버퍼 방식의 게임 제작 과정과 코드, 결과, 추가하고 싶은 기능들을 설명드리겠습니다.

<!-- outline-end -->
### 실제 코드
**코드의 대부분이 빠져 있고 변경된 핵심 부분들만 포스트에 작성했습니다.**    \n
때문에 전체 코드는 [깃허브 코드]()를 참고해 주시면 됩니다.

#### Sever
**Buffer_Channel_Server.java**{:data-align="center"}
```java
public void player(SocketChannel channel, int num, int[] board) throws InterruptedException {

    System.out.println(num);
    try {
        if (num == 0) {
            System.out.println(name + "님이 게임에서 떠났습니다");
            HelperMethods.sendMessage(client, "안녕히 가세요");
            client.close();
            informLeave(this);
        }

        if (this.rest == 1 && board[sum] == 3) {
            msg = "다음턴에 이동할 수 있습니다.";
            this.rest = 2;
        } else if (this.rest == 1) {
            this.rest = 0;
        } else {
            sum += num;
            if (sum >= board.length) {
                HelperMethods.sendMessage(channel, "win");
                channel.close();
                Buffer_Channel_Server.clients.removeIf(handler -> handler.name.equals(name));
                for (ClientHandler handler : Buffer_Channel_Server.clients) {
                    if (!handler.name.equals(name)) {
                        HelperMethods.sendMessage(handler.client, "lose");
                        handler.client.close();
                        Buffer_Channel_Server.clients.remove(handler);
                    }
                }
                return;
            }
        }

        if (catchs())
            return;

        if (board[sum] == 1) {
            sum += num;
            msg = num+"칸을 점프해 "+sum+"칸에 도착했습니다";
        } else if (board[sum] == 2) {
            sum -= num;
            msg = num+"칸 뒤로 가 "+sum+"칸에 도착했습니다";
        } else if (board[sum] == 3 && this.rest == 0) {
            msg = sum+"칸에 도착했지만 무인도라 한 턴 쉽니다";
            this.rest = 1;
        } else if (this.rest == 2) {
            this.rest += 1;
        } else if (this.rest == 3) {
            this.rest = 0;
            msg = num+"칸을 이동해 "+sum+"칸에 도착했습니다";
        } else {
            msg = num+"칸을 이동해 "+sum+"칸에 도착했습니다";
        }

        if (catchs(msg))
            return;
    } catch (IOException | InterruptedException ex) {
        throw new RuntimeException(ex);
    }

    HelperMethods.sendMessage(client, msg);
    for (ClientHandler handler : Buffer_Channel_Server.clients) {
        if (!handler.name.equals(name)) {
            HelperMethods.sendMessage(handler.client, name + "이/가 " + msg);
        }
    }
}
```
위의 코드와 같이 특정 시스템 메시지를 제외한 메시지들은 한글로 변경해 클라이언트가 바로 받아 띄울 수 있도록 조정했습니다.

#### Client
**Buffer_Channel_Client 중 전 포스트와 변경된 부분.java**{:data-align="center"}
```java
if (key.isReadable()) { // 1
    msg = HelperMethods.receiveMessage(channel);
    System.out.println(msg);
}

if (msg.equals("quit"))
    System.exit(0);     // 2
```
1. 시스템에서 토큰화도 필요없고, 토큰화 된 값을 가지고 어떤 출력을 해야 하는지 확인하는 것도 아니기 때문에 서버와 같이 간단하게 메시지 전달 코드를 변경하였습니다.
2. 프로그램을 단순히 멈추는 것이 아닌 종료하도록 변경했습니다.

#### HelperMethods
**Server 부분 HelperMethods.java**{:data-align="center"}
```java
public static void sendMessage(SocketChannel socketChannel, String message) {
    try {
        message = URLEncoder.encode(message, StandardCharsets.UTF_8);   // 1
        ByteBuffer buffer = ByteBuffer.allocate(message.length() + 1);
        buffer.put(message.getBytes());
        buffer.put((byte) 0x00);
        buffer.flip();
        while (buffer.hasRemaining()) {
            socketChannel.write(buffer);
        }
    } catch (IOException ex) {
        ex.printStackTrace();
    }
}
```
서버가 메시지를 보내는 부분입니다. 클라이언트의 경우 메시지를 영어로 보내기 때문에 바꾸지 않았습니다.
1. ASCII와 같은 문자 체계를 사용하는 HTML form 방식의 인코딩을 통해 데이터를 전송하도록 했습니다.

**Client 부분 HelperMethods.java**{:data-align="center"}
```java
public static String receiveMessage(SocketChannel socketChannel) {
    try {
        ByteBuffer byteBuffer = ByteBuffer.allocate(256);   // 1
        String message = "";
        while (socketChannel.read(byteBuffer) > 0) {
            char byteRead = 0x00;
            byteBuffer.flip();
            while (byteBuffer.hasRemaining()) {
                byteRead = (char) byteBuffer.get();
                if (byteRead == 0x00) {
                    break;
                }
                message += byteRead;
            }
            if (byteRead == 0x00) {
                break;
            }
            byteBuffer.clear();
        }
        message = URLDecoder.decode(message, StandardCharsets.UTF_8);   // 1
        return message;
    } catch (IOException ex) {
        ex.printStackTrace();
    }
    return "";
}
```
클라이언트가 메시지를 받는 부분입니다. 마찬가지 이유로 Client의 sendMessage는 변경되지 않았습니다.
1. 전송받은 HTML form 방식의 바이트 형식을 먼저 읽고, 해당 디코드를 진행해 한글을 주고 받을 수 있도록 진행했습니다.
    - 단점은 버퍼의 크기가 많이 커져야 했던 것입니다.

### 실행 결과
![tony의 변경된 코드 진행 화면](:/school/network/s-4.jpg){:data-align="center"}
코드를 변경하고, 정상적으로 게임이 진행되는 것을 확인하실 수 있습니다.

### 참고
#### 반복문이 없어 클라이언트에서 입력을 잘못하면 게임 진행이 안되는 오류 수정
**Buffer_Channel_Client.java의 일부분**{:data-align="center"}
```java
if ("Your Turn".equals(msg)) {
    while (true) {
        System.out.println("주사위를 굴리려면 y, 게임을 끝내려면 n을 입력해 주세요");
        String game = scanner.nextLine();
        if (game.equals("Y") || game.equals("y")) {
            diceNum = random.nextInt(1, 7);
            System.out.println("\n주사위 숫자는 " + diceNum + "입니다.");
            msg = name + "#" + diceNum;
            HelperMethods.sendMessage(channel, msg);
            break;
        } else if (game.equals("N") || game.equals("n")) {
            HelperMethods.sendMessage(channel, "tony#quit");
            msg = "quit";
            break;
        }
    }
    if (msg.equals("quit"))
        break;
}
```

#### 전체 코드 링크
1. [업그레이드한 버퍼/채널 서버 코드](https://github.com/Othkkartho/Board_Game_Server/tree/Buffer_Channel_Server_Improve)
2. [업그레이드한 버퍼/채널 클라이언트 코드](https://github.com/Othkkartho/Board_Game_Client/tree/Buffer_Channel_Client_Improve)
