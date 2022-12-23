---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: network
title: 버퍼/채널 방식으로 보드게임 만들기

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
date: 2022-12-23 20:00:00 +0900

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

### 게임 제작할때 고려한 것

1. 주사위는 6면체 주사위 1개로 진행합니다
2. 게임 서버는 클라이언트의 이름을 받아 등록하고, 게임을 진행해야 합니다
    - 기본적으 2개 이상의 클라이언트가 게임하는 것을 기본으로 합니다만 혼자서도 가능합니다.
3. 게임의 보드에는 4개의 이벤트가 존재합니다
    - Go는 기본적인 이벤트로 나온 주사위의 값만큼 이동합니다
    - Jump는 나온 주사위의 값만큼 한 번 더 이동하는 이벤트입니다.
    - Back는 나온 주사위의 값만큼 다시 뒤로 이동하는 이벤트입니다. 즉 원래 본인이 움직이기 전 위치로 이동합니다.
    - Skip은 무인도로 1턴을 휴식하는 이벤트입니다.
    - 이벤트가 1번 동작하면 catch 동작을 제외한 이벤트는 다시 일어나지 않습니다.
4. 이외에 상대 클라이언트와의 상호작용 이벤트 1개가 존재합니다
    - Catch는 상대 말을 잡았을 때 나오는 이벤트이고, 2가지 유형으로 제작할 계획입니다
    - 하나는 상대를 잡으면 상대가 처음으로 돌아가고, 그 칸의 이벤트를 진행합니다
    - 다른 하나는 상대방 말을 잡으면 상대가 처음으로 돌아가고, 그 칸의 이벤트를 무시합니다
5. 게임 보드의 경우 서버에서 그 크기를 결정합니다
6. 보드 안의 아이템의 경우 랜덤으로 그 값을 저장합니다
7. 클라이언트가 알아서 순서대로 입력하는 방식이 아닌 순서를 서버에서 지정해 주면 그 값을 받고 클라이언트가 사용자의 응답을 기다립니다.

* * *

읽으시기 전에 코드가 너무 길어져 설명이 필요 없는 부분이나 스트림과 겹친다고 생각되는 코드를 삭제했습니다. 자세한 코드는 [참고](#전체-코드-링크)에 깃허브 링크를 첨부해 두었으니 같이 확인 하셔도 괜찮습니다.

### 실제 코드
#### Sever
**Buffer_Channel_Server.java**{:data-align="center"}
```java
private static Selector acceptClient() {
    Selector selector = null;
    try {
        selector = Selector.open();     // 1
        ServerSocketChannel sschannel = ServerSocketChannel.open(); // 2

        sschannel.bind(new InetSocketAddress(5001));    // 3
        sschannel.configureBlocking(false);             // 4

        sschannel.register(selector, SelectionKey.OP_ACCEPT);   // 5
        board = ClientHandler.boardSetup();

        boolean open = false;

        while (true) {
            selector.select();      // 6
            Set<SelectionKey> keys = selector.selectedKeys();   // 7
            Iterator<SelectionKey> iterator = keys.iterator();  // 8

            while (iterator.hasNext()) {            // 9
                SelectionKey key = iterator.next(); // 9

                if (key.isAcceptable()) {           // 10
                    SocketChannel client = sschannel.accept();  // 11
                    client.configureBlocking(false);    // 4
                    client.register(selector, SelectionKey.OP_READ);    // 5
                }

                if (key.isReadable()) { // 10
                    SocketChannel client = (SocketChannel) key.channel();   // 12
                    String received = HelperMethods.receiveMessage(client); // 13

                    StringTokenizer tokenizer = new StringTokenizer(received, "#"); // 14
                    String what = tokenizer.nextToken();
                    String data = tokenizer.nextToken();

                    if (what.equals("name")) {
                        ClientHandler handler = new ClientHandler(client, data, 0);
                        informNew(data);
                        clients.add(handler);
                        if (clients.size() == 1) {  // 15
                            HelperMethods.sendMessage(handler.client, "host");
                            TimeUnit.SECONDS.sleep(1);
                        }
                    }

                    if (what.equals("open")) {
                        open = true;
                    }
                }
                iterator.remove();      // 16
            }
            if (open == true) {
                break;
            }
        }
    } catch (IOException e) {
        e.printStackTrace();
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
    return selector;
}
```
해당 메서드는 게임이 실행되지 전에 클라이언트들의 요청을 받고, 클라이언트를 저장하는 메서드입니다.
1. Selector를 열어 새로운 Selector를 만들고 변수에 저장합니다.
2. 인터넷 프로토콜 소켓의 서버소켓채널을 열어 새로운 ServerSocketChannel을 만들고 변수에 저장합니다.
3. 입력한 포트 번호를 가지고 와일드카드 주소(0:0:0:0:0:0:0:0)+포트 번호인 소켓 주소를 만듭니다.
4. 서버소켓채널이 Selector로 동작하기 위해 해당 Blocking 설정을 끕니다.
5. selector에 해당 채널을 SelectionKey로 해당 Selector가 잡을 동작을 설정합니다.
6. 해당 채널에 등록한 동작 요청이 들어올 때 까지 Block 하고 있습니다.
7. selector에서 동작과 관련되 주석 5번에서 설정했던 값을 불러와 중복이 불가능한 Set에 저장합니다.
8. Set에 저장된 값을 포인터와 같은 역할을 하는 순환자(Iterator)에 저장해 반복문을 돌려 해당 동작을 찾도록 합니다.
9. Iterator에 저장된 다음 값을 불러와 해당 key 값을 저장합니다.
10. 해당 채널에 들어온 요청이 accept인지, read인지를 확인하고, 해당하는 조건문을 실행합니다.
11. accept 요청을 한 클라이언트의 정보를 저장합니다.
12. Read 요청을 보낸 클라이언트의 정보를 key에서 가져와 변수에 저장합니다.
13. 클라이언트에서 보낸 정보를 받아 변수에 저장합니다.(자세한 사항은 해당 동작이 진행되는 HelperMethods 클래스에서 설명하겠습니다)
14. 받아온 문자열을 설정한 값에 맞춰 토큰화 시켜 각 변수에 저장합니다.
15. 만약 클라이언트가 1개라면 호스트라는 메시지를 보내 클라이언트가 관련된 동작을 수행하도록 합니다.
16. iterator의 값을 삭제하지 않으면 이후 동작에서 null 클라이언트를 가져와 NullPointerException을 던지고 서버가 죽어 반드시 실행해야 합니다.

**Buffer_Channel_Server.java**{:data-align="center"}
```java
private static void startGame(Selector selector) throws InterruptedException {
    for (ClientHandler handler : clients) {                     // 1
        HelperMethods.sendMessage(handler.client, "Game Start");
        TimeUnit.SECONDS.sleep(1);
    }

    try {
        int i = 0;

        HelperMethods.sendMessage(clients.get(i).client, "Your Turn");  // 2
        TimeUnit.SECONDS.sleep(1);

        while (true) {
            selector.select();
            Set<SelectionKey> keys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = keys.iterator();

            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();

                if (key.isReadable()) {
                    SocketChannel client = (SocketChannel) key.channel();
                    String received = HelperMethods.receiveMessage(client);

                    System.out.println("Received: " + received);

                    StringTokenizer tokenizer = new StringTokenizer(received, "#");
                    String what = tokenizer.nextToken();
                    String data = tokenizer.nextToken();

                    if (what.equals("name")) {          // 3
                        HelperMethods.sendMessage(client, "Unable in the game.");
                    } else if (data.equals("quit")) {   // 4
                        clients.removeIf(handler -> handler.name.equals(what));
                        for (ClientHandler handler : clients)
                            if (!handler.name.equals(what)) {
                                HelperMethods.sendMessage(handler.client, what+" is leaved");
                                TimeUnit.SECONDS.sleep(1);
                            }
                        client.close();
                    } else {        // 5
                        int diceNum = Integer.parseInt(data);   // 5

                        for (ClientHandler handler : clients) {
                            if (handler.name.equals(what)) {
                                handler.player(handler.client, diceNum, board); // 6
                            }
                        }
                    }
                }
                iterator.remove();
            }

            if (clients.size() != 0) {      // 2
                i = (i+1)% clients.size();
                HelperMethods.sendMessage(clients.get(i).client, "Your Turn");
                TimeUnit.SECONDS.sleep(1);
            }
        }
    } catch (IOException e) {
        e.printStackTrace();
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
}
```
위의 클라이언트 등록을 종료하고, 게임을 시작하는 메서드입니다.
1. 게임이 시작되면 등록된 클라이언트 전원에 게임 시작 알림을 보냅니다.
2. 가장 처음 등록한 호스트를 먼저 본인의 턴이라는 것을 알려주고, 사용자가 게임을 진행할 수 있도록 합니다. 그 이후로는 등록된 클라이언트들을 돌아가며 게임을 진행할 수 있도록 합니다.
3. 만약 이름이 온다면 설정된 시간 내에 Accept되긴 했지만 이름을 등록하지 않은 사람들이 이름을 등록할 경우 메시지를 보내고 등록되지 않습니다.
4. 만약 메시지가 quit가 나오면 클라이언트가 종료했다는 의미이므로 해당 인원을 벡터에서 제거하고, 다른 클라이언트들에게 해당 클라이언트가 종료했다고 알리며, 게임을 계속 진행시킵니다.
5. 아니면 게임이 진행되고 있다는 의미이므로 해당 값을 주사위 숫자로 파악하고, 변수에 저장합니다.
6. 주사위 값의 결과를 판단합니다.

* * *

**ClientHandler.java**{:data-align="center"}
```java
public void player(SocketChannel channel, int num, int[] board) throws InterruptedException {

    System.out.println(num);
    try {
        if (this.rest == 1 && board[sum] == 3) {    // 1
            msg = "skip2#0";
            this.rest = 2;
        } else if (this.rest == 1) {
            this.rest = 0;
        } else {
            sum += num;
            if (sum >= board.length) {
                HelperMethods.sendMessage(channel, "win#" + board.length);
                channel.close();
                for (ClientHandler handler : Buffer_Channel_Server.clients) {
                    if (!handler.name.equals(name)) {
                        HelperMethods.sendMessage(handler.client, "lose#"+handler.sum);
                        handler.client.close();
                        TimeUnit.SECONDS.sleep(1);
                    }
                }
            }
        }

        if (catchs())   // 2
            return;

        if (board[sum] == 1) {  // 3
            sum += num;
            msg = "jump#"+sum+"#"+num;
        } else if (board[sum] == 2) {
            sum -= num;
            msg = "back#"+sum+"#"+num;
        } else if (board[sum] == 3 && this.rest == 0) {
            msg = "skip#"+sum;
            this.rest = 1;
        } else if (this.rest == 2) {
            this.rest += 1;
        } else if (this.rest == 3) {
            this.rest = 0;
            msg = "go#"+sum+"#"+num;
        } else {
            msg = "go#"+sum+"#"+num;
        }

        if (catchs(msg))    // 2
            return;
    } catch (IOException | InterruptedException ex) {
        throw new RuntimeException(ex);
    }

    HelperMethods.sendMessage(client, msg); // 4
    TimeUnit.SECONDS.sleep(1);
    for (ClientHandler handler : Buffer_Channel_Server.clients) {   // 5
        if (!handler.name.equals(name)) {
            HelperMethods.sendMessage(handler.client, name + "#" + msg);
            TimeUnit.SECONDS.sleep(1);
        }
    }
}
```
플레이어 주사위의 결과를 확인하는 메서드입니다.
1. 해당 플레이어가 무인도에 있는지 없는지를 판단해 해당하는 조건문을 작동시킵니다.
2. 해당 플레이어가 상대 말을 잡았는지를 확인합니다.
3. 플레이어가 위차한 칸의 어떤 아이템이 있는지 판단해 클라이언트가 관련 메시지를 출력할 수 있도록 메시지를 만들어 변수에 저장합니다.
4. 모든 결과가 포함된 메시지를 플레이어에게 전달합니다.
5. 플레이하고 있는 유저의 결과 메시지를 다른 플레이어들에게 전달합니다.

**ClientHandler.java**{:data-align="center"}
```java
public boolean catchs() throws InterruptedException {
    for (ClientHandler handler : Buffer_Channel_Server.clients) {
        if (handler.name.equals(name))
            continue;
        if (sum > 0 && handler.sum > 0 && handler.sum == sum) {
            handler.sum = 0;
            HelperMethods.sendMessage(client, "catch#"+sum);
            TimeUnit.SECONDS.sleep(1);
            HelperMethods.sendMessage(handler.client, name+"#catch#"+sum);
            return true;
        }
    }
    return false;
}

public boolean catchs(String msg) throws InterruptedException {
    for (ClientHandler handler : Buffer_Channel_Server.clients) {
        if (handler.name.equals(name))
            continue;
        if (sum > 0 && handler.sum > 0 && handler.sum == sum) {
            handler.sum = 0;
            HelperMethods.sendMessage(client, "catch#"+msg);
            TimeUnit.SECONDS.sleep(1);
            HelperMethods.sendMessage(handler.client, name+"#catch");
            return true;
        }
    }
    return false;
}

public static int[] board(int[] board) {
    // Stream과 코드가 같습니다.
}
```
플레이어가 상대 말을 언제 잡았는지에 따라 전달해야 하는 메시지가 다르기 때문에 관련된 메서드를 호출해 메시지를 만들고 전달합니다.

#### Client
**Buffer_Channel_Client.java**{:data-align="center"}
```java
public Buffer_Channel_Client() {

    try {
        Selector selector = Selector.open();
        SocketChannel channel = SocketChannel.open(new InetSocketAddress("localhost", 5001));   // 1

        TimerTask task = new TimerTask() {  // 2
            public void run() {
                HelperMethods.sendMessage(channel, "open#null");
            }
        };

        System.out.println("Connected to Chat Server");
        System.out.print("Name: ");
        String name = scanner.nextLine();
        msg = "name#"+name;
        HelperMethods.sendMessage(channel, msg);

        channel.configureBlocking(false);
        channel.register(selector, SelectionKey.OP_READ);

        while (true) {
            selector.select();
            Set<SelectionKey> keys = selector.selectedKeys();
            Iterator<SelectionKey> it = keys.iterator();

            while (it.hasNext()) {
                SelectionKey key = it.next();

                if (key.isReadable()) {     // 3
                    msgs = PrintIt.receive(channel);
                    msg = msgs[0];
                    PrintIt.check(msgs, channel);
                }

                if ("host".equals(msg)) {   // 2
                    Timer timer = new Timer("Timer");
                    long delay = 20000L;    // 20초
                    timer.schedule(task, delay);
                }

                if ("Your Turn".equals(msg)) {
                    System.out.println("주사위를 굴리려면 y, 끝내려면 n을 입력해 주세요");
                    String game = scanner.nextLine();
                    if (game.equals("Y") || game.equals("y")) {
                        diceNum = random.nextInt(1, 7);
                        System.out.println("\n주사위 숫자는 " + diceNum + "입니다.");
                        msg = name + "#" + String.valueOf(diceNum);
                        HelperMethods.sendMessage(channel, msg);
                    } else if (game.equals("N") || game.equals("n")) {
                        HelperMethods.sendMessage(channel, "tony#quit");
                        msg = "quit";
                        break;
                    } else {
                        System.out.println("주사위를 굴리려면 y, 끝내려면 n을 입력해 주세요");
                    }
                }

                if ("win".equals(msg)) {
                    msg = "quit";
                    System.out.println("이겼습니다.");
                } else if ("lose".equals(msg)) {
                    msg = "quit";
                    System.out.println("졌습니다.");
                }
                it.remove();
            }
            if (msg.equals("quit"))
                break;
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
클라이언트의 실제 대부분의 동작이 일어나는 메서드입니다.
1. 서버에 접근을 시도합니다.
2. 일정 시간이 지난 후 호스트가 서버에 게임을 진행해도 좋다는 메시지를 보내기 위한 코드입니다.
3. 서버에게서 받은 메시지를 토큰화 시켜 해당 메시지에 맞는 동작을 진행시키기 위해 토큰화 시키는 코드입니다.

* * *

**PrintIt.java**{:data-align="center"}
```java
 public static String[] receive(SocketChannel channel) {
    String msg = HelperMethods.receiveMessage(channel);

    StringTokenizer tokenizer = new StringTokenizer(msg, "#");
    String what = null, data1 = null, data2 = null, data3 = null, data4 = null;
    try {
        what = tokenizer.nextToken();
        data1 = tokenizer.nextToken();
        data2 = tokenizer.nextToken();
        data3 = tokenizer.nextToken();
        data4 = tokenizer.nextToken();
    } catch (Exception ignored) {}

    String[] msgs = {what, data1, data2, data3, data4};

    return msgs;
}
```
실질적인 토큰화를 진행하고, 토큰화된 값들을 문자열 리스트의 형태로 return시킵니다.

**PrintIt.java**{:data-align="center"}
```java
public static void check(String[] msgs, SocketChannel channel) {
    if (msgs[1] != null) {
        switch (msgs[0]) {
            case "jump":
                System.out.println(msgs[2] + "칸을 점프해 " + msgs[1] + "칸에 도착했습니다.");
                break;
            case "back":
                System.out.println(msgs[2] + "칸 뒤로가 " + msgs[1] + "칸에 도착했습니다.");
                break;
            case "skip":
                System.out.println(msgs[1] + "칸에 도착했지만 무인도에 도착해 한 턴 휴식합니다.");
                break;
            case "skip2":
                System.out.println("다음턴에 이동할 수 있습니다.");
                break;
        }
    } else {
        System.out.println(msgs[0]);
    }
}
```
입력받은 결과에 맞는 메시지를 콘솔에 출력시킵니다. 모든 코드는 아니고 코드의 일부분입니다. 전체 코드는 [깃허브](https://github.com/Othkkartho/Board_Game_Server/tree/Buffer_Channel_Server)에서 확인하실 수 있습니다.

#### HelperMethods
**ClientHandler.java**{:data-align="center"}
```java
public static void sendMessage(SocketChannel socketChannel, String message) {
    try {
        ByteBuffer buffer = ByteBuffer.allocate(message.length() + 1);  // 1
        buffer.put(message.getBytes());     // 2
        buffer.put((byte) 0x00);            // 2
        buffer.flip();                      // 2
        while (buffer.hasRemaining()) {     // 3
            socketChannel.write(buffer);
        }
    } catch (IOException ex) {
        ex.printStackTrace();
    }
}
```
데이터를 원하는 소켓 채널에 전달할 때 사용하는 메서드입니다.
1. 전달받은 메시지에 1을 추가해 바이트 버퍼를 생성합니다.
    - 참고로 바이트에서 문자열을 만드는 이 메서드는 바이트->ASCII로 동작하기 때문에 바이트의 크기를 늘린다고 UTF-8과 같은 문자표로 인식하지 못하고 글자가 깨집니다.
2. 입력받은 메시지를 바이트로 바꿔 버퍼에 저장하고 종료를 나타내는 0x00(0)을 바이트에 넣고, 서버 소켓이 해당 버퍼를 읽고 전송할 수 있도록 flip를 실행시켜 정상적으로 값이 전달되도록 합니다.
3. 버퍼를 Position과 Limit 사이에 요소가 없을 때 까지 버퍼의 값을 해당 소켓 채널에 전달합니다.

**ClientHandler.java**{:data-align="center"}
```java
public static String receiveMessage(SocketChannel socketChannel) {
    try {
        ByteBuffer byteBuffer = ByteBuffer.allocate(16);    // 1
        String message = "";                                // 2
        while (socketChannel.read(byteBuffer) > 0) {        // 3
            char byteRead = 0x00;
            byteBuffer.flip();                      // 4
            while (byteBuffer.hasRemaining()) {
                byteRead = (char) byteBuffer.get(); // 5
                if (byteRead == 0x00) {             // 6
                    break;
                }
                message += byteRead;                // 5
            }
            if (byteRead == 0x00) {
                break;
            }
            byteBuffer.clear();                     // 7
        }
        return message;
    } catch (IOException ex) {
        ex.printStackTrace();
    }
    return "";
}
```
해당 소켓 채널에서 전달한 데이터를 받을 때 사용하는 메서드입니다.
1. 데이터를 받기위한 바이트버퍼를 설정하고 싶은 크기에 맞춰 생성합니다.
2. 전달된 메시지를 저장할 변수를 선언합니다.
3. 해당 소켓채널에서 전달된 데이터를 읽는 작업을 진행하고, 전달받은 데이터가 존재하면 반복문을 실행합니다.
4. Limit는 Position(현재 위치)에 설정되고, Position은 0에 설정되는 버퍼 read, write전에 수행하는 메서드입니다.
5. 전달받은 데이터들을 처음부터 하나씩 꺼내 byteRead에 추가하고, pos값을 그 다음으로 넘겨 다음 값을 읽을 준비를 합니다. 읽은 값은 문자열에 추가합니다.
6. 읽은 값이 0x00이면 문자를 보낼때 해당 바이트를 종료 바이트로 설정해 넣어 놨기 때문에 이 뒤는 0으로 판단 더 읽을 값이 없다고 생각하고, 읽기를 종료합니다.
7. 다음 전달되는 값을 읽기 위해 버퍼를 초기화 시킵니다.

### 실행 결과
![tony의 게임 진행 화면](:/school/network/bc-1.jpg){:data-align="center"}
과제를 위해 3개를 다 찍었었지만 해당 포스트에서는 tony 화면만 보여드리겠습니다.
1. 보시면 tony가 들어오고 가장 먼저 들어왔기 때문에 host를 받습니다. 그 이후 tom과 susan이 들어오고 일정 시간 뒤에 open을 보내며 게임을 시작합니다.
2. 맨 처음 게임을 시작하고, 계속해서 본인과 상대 메시지가 정상적으로 출력 되는 것을 확인하실 수 있습니다.
3. 끝나면 tony는 25칸을 먼저 넘었기 때문에 이겼다는 메시지가 출력되고, 다른 클라이언트들은 졌습니다를 출력합니다.

![tony의 catch 이벤트 발생](:/school/network/s-4.jpg){:data-align="center"}
tony의 catch 이벤트 발생 메시지가 정상적으로 출력되고, 마지막으로 돌아 간 것을 확인하실 수 있습니다.

![tony는 게임을 종료함](:/school/network/s-6.jpg){:data-align="center"}

![tony가 게임 종료했다는 메시지가 출력되고 게임이 정상적으로 진행됨](:/school/network/s-7.jpg){:data-align="center"}
1. tony가 n을 눌러 게임을 종료합니다.
2. tom의 화면에 tony가 게임을 종료했다는 메시지가 뜨고, Your Turn이 계속적으로 뜨며 게임이 정상적으로 진행되는 것을 확인하실 수 있습니다.

### 참고
#### 읽기 작업 전 filp가 실행되어야 하는 이유
사실 위에 flip이 실행될 때마다 왜 flip이 동작해야 하는 지 간단한 이유는 설명을 했지만 명확하게 이해 되지 않아 추가적인 설명을 위해 작성합니다.   \n

클라이언트가 본인의 이름을 전달했을 때를 예로 들어보겠습니다.
1. 서버는 name#tony라는 값을 바이트로 받습니다. [110,97,109,101,35,116,111,110,121,0,0,0,0,0,0] 의 값이 버퍼에 저장됩니다.
2. 그럼 해당 버퍼는 pos(현재 위치, Position)값이 10이 되고, lim(Limit) 값이 16, cop가 16으로 설정되어 있습니다. 이유는 해당 버퍼의 끝을 이미 16으로 설정했고, 입력된 값이 총 9바이트이기 때문에 그 다음인 10을 pos가 가리키고 있는 것입니다.
3. 그럼 만약 이 상태에서 그냥 실행한다면 버퍼는 pos가 가리키고 있는 위치부터 읽게 됩니다. 따라서 값을 읽거나 쓰기 전에 flip을 해 놔야 시작값과 마지막 값이 사용자가 생각하는 대로 위치되 정상적으로 값을 읽거나 쓰게 됩니다.
    - 값을 추가할 때는 안쓰긴 하겠네요.

#### 전체 코드 링크
1. [버퍼/채널 클라이언트 코드](https://github.com/Othkkartho/Board_Game_Client/tree/Buffer_Channel_Client)
2. [버퍼/채널 서버 코드](https://github.com/Othkkartho/Board_Game_Server/tree/Buffer_Channel_Server)