---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: network
title: 스트림 방식의 보드게임 제작하기

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
date: 2022-12-21 20:00:00 +0900

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
해당 코드는 [참고](#참고)에 있습니다.

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
```java
public class StreamServer {
    public static Vector<ClientHandler> clients = new Vector<>();   // 10

    private static void informNew(String name) throws IOException { // 7
        for (ClientHandler handler : clients)
            handler.dos.writeUTF(name + " is just logged in");
    }

    public static void main(String[] args) {
        try {
            ServerSocket server = new ServerSocket(3005);   // 1
            int[] board = new int[25];                      // 2
            ClientHandler.board(board);                     // 3

            while (true) {
                System.out.println("Server is waiting");
                Socket socket = server.accept();            // 4
                System.out.println("client is connected: " + socket);

                DataInputStream dis = new DataInputStream(socket.getInputStream());     // 5
                DataOutputStream dos = new DataOutputStream(socket.getOutputStream());  // 5
                String name = dis.readUTF();        // 6
                System.out.println(name + ": Welcome to the server.");
                informNew(name);                    // 7

                ClientHandler handler = new ClientHandler(socket, dis, dos, name, 0, board);    // 8
                Thread thread = new Thread(handler);    // 9
                System.out.println("Adding this client to client vector");
                clients.add(handler);   // 10
                thread.start();         // 11
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
1. 서버를 입력한 포트번호에 맞춰 초기화합니다.
2. 서버에서 진행 될 게임의 아이템들을 보드에 등록 시키기 위해 보드 변수를 생성합니다.
3. 아래에서 다룰 ClientHandler안의 board를 만드는 함수를 호출해 보드를 만듭니다.
4. 주석 1번에서 입력한 포트의 번호대로 서버를 실행시키고, 클라이언트의 접근 요청을 받을 준비를 합니다. 해당 동작은 Blocking으로 동작합니다.
5. 클라이언트가 접슥을 하면 해당 소켓이 변수로 저장되고, 해당 소켓의 Stream으로 서버서버가 생성한 스트림 변수를 초기화해 해당 소켓으로의 Input, Output 명령을 수행할 수 있도록 합니다.
6. 사용자가 접속하면 사용자의 등록을 위해 이름을 입력받습니다.
7. 만약 해당 사용자보다 일찍 들어온 사용자들이 있다면 그 사용자들에게 현재 접속한 플레이어를 알려줍니다.
8. 소켓, 입출력을 위한 스트림, 이름, 그 사용자의 최종 위치인 SUM 변수, 생성된 board로 클라이언트 핸들러를 초기화 합니다.
9. 해당 클라이언트와의 통신을 위해 서버가 새로운 스레드를 초기화시키며 생성합니다.
10. 클라이언트의 정보를 저장하기 위해 동기화가 특징인 Vector에 클라이언트의 정보를 저장합니다.
11. 최종적으로 클라이언트와의 통신을 위해 스레드를 동작시킵니다.
   
위의 단계를 지속적으로 반복하며 클라이언트를 받아들이는 역할을 이 클레스에서는 진행합니다.


**ClientHandler.java 주요 코드**{:data-align="center"}
```java
@Override
public void run() {         // 1
    int diceNum;            // 2
    boolean rest = false;   // 2
    String msg;             // 2
    String data;            // 2

    while (true) {
        try {
            diceNum = dis.read();   // 3
            System.out.println(diceNum);
            if (diceNum == 0) {     // 4
                System.out.println(this.name + " is just leaved.");
                this.s.close(); // 접속을 종료하는 클라이언트의 서버 쪽 소켓을 닫음
                informLeave(this);
                break;
            }

            if (rest == true) {     // 5
                msg = "다음 턴에 이동할 수 있습니다.";
                rest = false;
            }
            else {
                this.sum += diceNum;

                checkEnd(board);        // 8
                catchs(name, sum, dos); // 7

                if (board[this.sum] == 1) {
                    this.sum += diceNum;
                    msg = "Jump!! " + diceNum + "칸을 점프해 현재 " + this.sum + "칸 입니다.";
                }
                else if (board[this.sum] == 2) {
                    this.sum -= diceNum;
                    msg = "Back!! " + diceNum + "칸을 후퇴해 현재 " + this.sum + "칸 입니다.";
                }
                else if (board[this.sum] == 3) {
                    msg = "현재 " + this.sum + "칸 입니다.\n무인도에 걸려 한턴을 쉽니다.";
                    rest = true;
                }
                else {
                    msg = "현재 " + this.sum + "칸 입니다.";
                }

                checkEnd(board);    // 8
            }
            dos.writeUTF(msg);      // 6
            catchs(name, sum, dos); // 7

            for (ClientHandler handler : StreamServer.clients) {    // 9
                if (!handler.name.equals(this.name)) {
                    data = this.name + "는 " + msg;
                    handler.dos.writeUTF(data);
                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}

private void checkEnd(int[] board) throws IOException {     // 8
    if (this.sum >= board.length) {
        for (ClientHandler handler : StreamServer.clients) {
            if (handler.name.equals(this.name)) {
                dos.writeUTF("win");
                this.s.close();
            }
            else {
                handler.dos.writeUTF("lose");
                handler.s.close();
            }
        }
    }
}

private static void catchs(String name, int sum, DataOutputStream dos) throws IOException { // 7
    for (ClientHandler handler : StreamServer.clients) {
        if (handler.name.equals(name))
            continue;
        if (sum > 0 && handler.sum > 0 && handler.sum == sum) {
            handler.sum = 0;
            dos.writeUTF("상대방의 말을 잡아 상대가 처음으로 돌아갑니다.");
            handler.dos.writeUTF(name + "에게 말이 잡혀 처음으로 돌아갑니다.");
        }
    }
}
```
1. 스레드가 실행되면 해당 메소드가 동작합니다.
2. 각각 위에서 부터 방금 나온 주사위 숫자, 무인도에 걸렸는지 안걸렸는지를 판단하는 변수, 전달할 메시지를 저장하는 msg와 data 변수를 지정했습니다.
3. 클라이언트에서 보낸 int형의 주사위 정보를 int 형식으로 받습니다.
4. 해당 주사위의 숫자가 0이면 클라이언트가 접속을 종료한다는 의미이기 때문에 해당 프로세스를 진행합니다.
5. 해당 플레이어가 무인도에 걸렸는지를 확인하고 걸렸으면 해당 프로세스를 진행
6. 위에서 나온 결과를 msg에 저장하고 그 값을 클라이언트에게 전달합니다.
7. 해당 클라이언트가 이동한 후 상대 말을 잡았는지 아닌지 전체를 돌며 조사 후 만약 잡았으면 해당 내용을 전달하고, 잡힌 상대 말을 처음 위치로 돌립니다.
8. 해당 플래이어의 말이 보드의 크기 이상에 있다면 해당 게임이 종료된 것이므로 그 여부를 확인하고, 만약 종료되었으면 그 결과를 클라이언트들에 통보하고, 해당 소켓을 종료시킵니다.
9. 다른 플레이어들도 다른 플래이어가 어떤 결과인지 알게 하기 위해 다른 플레이어들을 찾아 해당 클라이언트의 결과를 전달합니다. 

**ClientHandler.java 이외 코드**{:data-align="center"}
```java
public class ClientHandler implements Runnable {
    Socket s;
    DataInputStream dis;
    DataOutputStream dos;
    String name;
    int sum;
    int[] board;

    public ClientHandler(Socket s, DataInputStream dis, DataOutputStream dos, String name, int sum, int[] board) {  // 1
        this.s = s;
        this.dis = dis;
        this.dos = dos;
        this.name = name;
        this.sum = sum;
        this.board = board;
    }

    private void informLeave(ClientHandler handler) throws IOException {    // 2
        for (ClientHandler mc : StreamServer.clients) {
            if (!mc.name.equals(handler.name)) {    // 로그아웃하는 클라이언트가 아니면
                mc.dos.writeUTF(handler.name + " is just leaved.");
            }
        }
    }

    @Override
    public void run() {
        // 위에 설명을 진행해 코드를 제외함
    }

    private void checkEnd(int[] board) throws IOException {
        // 위에 설명을 진행해 코드를 제외함
    }

    private static void catchs(String name, int sum, DataOutputStream dos) throws IOException {
        // 위에 설명을 진행해 코드를 제외함
    }

    public static int[] board(int[] board) {    // 3
        for (int n : board)
            board[n] = 0;
        int i = 0;
        Random random = new Random();

        while (true) {
//            i += random.nextInt(board.length/20, board.length/10);
            i += 3;
            if (i >= board.length)
                break;
            board[i] = 1;   // jump
        }

        i = 0;
        while (true) {
//            i += random.nextInt(board.length/20, board.length/10);
            i += 4;
            if (i >= board.length)
                break;
            if (board[i] == 0)
                board[i] = 2;   // back
        }

        i = 0;
        while (true) {
//            i += random.nextInt(board.length/10, board.length/5);
            i += 5;
            if (i >= board.length)
                break;
            if (board[i] == 0)
                board[i] = 3;   // skip
        }
        return board;
    }
}
```
1. 클라이언트를 저장하기 위해 해당 변수 초기화 후 값을 저장하기 위한 메소드입니다.
2. 다른 클라이언트가 로그아웃 시 그 결과를 다른 클라이언트들에 알리기 위한 메소드입니다.
3. 보드를 만드는 메소드입니다. 원래는 Random으로 진행해 게임에 우연성을 추가하려 했지만, 과제 제출시 필요한 테스트의 편의성 때문에 고정 값으로 변경했습니다.

#### Client
**StreamClient.java 스레드 코드**{:data-align="center"}
```java
Thread sendMessage = new Thread(() -> {         // 1
    System.out.println("주사위를 굴리려면 y, 끝내려면 n을 입력해 주세요");
//            int diceNum = 2;
    while (true) {
        String game = sc.nextLine();

        if (game.equals("Y") || game.equals("y")) {
            int diceNum = random.nextInt(1, 7);
//                    diceNum += 1;
            System.out.println("\n주사위 숫자는 " + diceNum + "입니다.");
            try {
                dos.write(diceNum);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        else if (game.equals("N") || game.equals("n")) {
            try {
                dos.write(0);           // 2
                break;
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        else {
            System.out.println("주사위를 굴리려면 y, 끝내려면 n을 입력해 주세요");
        }
    }

    try {
        dis.close();        // 3
        dos.close();        // 3
        socket.close();     // 3
    } catch (IOException e) {
        e.printStackTrace();
    }
});

Thread readMessage = new Thread(() -> { // 4
    while (true) {
        try {
            String msg = dis.readUTF();
            if (msg.equals("win")) {
                System.out.println("게임에서 승리하였습니다.\n접속을 종료합니다.");
                break;
            }
            else if (msg.equals("lose")) {
                System.out.println("게임에서 졌습니다.\n접속을 종료합니다.");
                break;
            }
            System.out.println(msg + "\n");
        } catch (IOException e1) {
            try {
                socket.close(); // 3
                dis.close();    // 3
                dos.close();    // 3
                break;
            } catch (IOException e2) {
                e2.printStackTrace();
            }
            e1.printStackTrace();
        }
    }
    try {
        socket.close();         // 3
        dis.close();            // 3
        dos.close();            // 3
    } catch (IOException e2) {
        e2.printStackTrace();
    }
});

sendMessage.start();        // 5
readMessage.start();        // 5
```
1. 클라이언트가 서버에서 데이터를 받기 위해 사용될 스레드를 정의합니다.
2. 클라이언트 종료를 통보하기 위해 서버에 0을 보내 서보도 해당 클라이언트의 소켓을 종료하게 합니다.
3. 데이터 Input/Output 스트림과 소켓을 종료해 서버와의 접속을 종료합니다.
4. 서버에 데이터를 보내기 위한 스레드를 정의합니다.
5. 서버와의 통신을 위해 만든 2개의 스레드를 동작시킵니다.

**StreamClient.java 스레드를 제외한 전체 코드**{:data-align="center"}
```java
public class StreamClient {
    final static int ServerPort = 3005;     // 1

    public static void main(String[] args) throws IOException {
        Scanner sc = new Scanner(System.in);
        InetAddress ip = InetAddress.getByName("localhost");    // 1
        Random random = new Random();

        Socket socket = new Socket(ip, ServerPort);     // 1
        System.out.println("Client is connected to the chat server");

        DataInputStream dis = new DataInputStream(socket.getInputStream());
        DataOutputStream dos = new DataOutputStream(socket.getOutputStream());

        System.out.print("Name: ");
        String name = sc.nextLine();
        dos.writeUTF(name);

        Thread sendMessage = new Thread(() -> {
            // 위에 설명을 진행해 코드를 제외함
        });

        Thread readMessage = new Thread(() -> {
            // 위에 설명을 진행해 코드를 제외함
        });

        sendMessage.start();
        readMessage.start();
    }
}
```
1. 서버의 포트와 서버의 IP 주소를 찾아와 소켓을 만들어 서버와 통신할 준비를 합니다.
나머지 코드는 서버와 유사합니다.

### 실행 결과
#### 진행 설명
1. 서버를 실행 시키고 1명 이상의 플레이어가 들어와 이름을 등록하면 게임이 진행될 수 있습니다.
2. 유저가 y를 누른다면 게임은 진행됩니다.
3. 주사위는 6면체 주사위 1개로 이루어져 있고, 주사위 값을 서버에 전송합니다.
4. 전송된 값으로 서버는 기존 클라이언트의 거리 값에 해당 주사위 값을 더해 보드판의 위치를 찾습니다.
5. 해당 위치에 맞는 이벤트가 진행됩니다.
    - Jump는 해당 주사위의 수 만큼 앞으로 이동합니다.
    - Back은 해당 주사위의 수 만큼 뒤로 이동합니다.
    - Skip은 한 턴을 쉬어야 나올 수 있습니다.
6. 이벤트의 진행 전과 후 catch 이벤트를 조사해 만약 상대편 말을 잡았다면 잡았다는 메시지를 보내고, 상대방 말을 처음 시작위치로 돌려 보냅니다.
7. catch 이벤트나, Jump, Back 이벤트가 일어난 후에는 Catch 이벤트를 제외한 모든 이벤트가 진행되지 않습니다. 즉 해당 칸의 아이템이 무엇인지 고려하지 않습니다.
8. 위의 과정을 지속하다가 만약 한 사람이 보드의 크기를 넘게 된다면 해당 인원에게 Win을 보내고, 진 사람에게는 lose를 보내고 세션을 닫습니다.

#### 실제 진행 사진
![jump 이벤트 발생](:/school/network/s-jump.jpg){:data-align="center"}

![back 이벤트 발생](:/school/network/s-back.jpg){:data-align="center"}

![skip 이벤트 발생](:/school/network/s-skip.jpg){:data-align="center"}
위 3개의 이벤트 모두 해당 블록을 밟았을 때 이벤트가 정상적으로 발생되는 것을 확인하실 수 있습니다.

![catch 이벤트 발생](:/school/network/s-catch.jpg){:data-align="center"}
상대편 말을 잡았을 때 해당 이벤트가 정상적으로 작동하고, 상대편 말이 처음으로 돌아간 것을 확인하실 수 있습니다.

![중간 종료 후 정상적으로 종료됨](:/school/network/s-end.jpg){:data-align="center"}

![상대 클라이언트가 종료된 것을 확인할 수 있음](:/school/network/s-end2.jpg){:data-align="center"}
중간에 종료했을 때 종료한 클라이언트는 종료가 되고, 다른 클라이언트에게 해당 내용이 전달되는 것을 확인하실 수 있습니다.

![게임이 끝났을 때 win 출력](:/school/network/s-win.jpg){:data-align="center"}

![게임이 끝났을 때 lose 출력](:/school/network/s-lose.jpg){:data-align="center"}
한 클라이언트가 보드 이상의 칸에 도달했을 때 win과 lose에 대한 출력이 정상적으로 이루어 지고, 종료된 것을 확인하실 수 있습니다.


### 변경사항
#### 상대를 잡으면 해당 칸의 이벤트는 무시하게 변경
사실 상대를 잡으면 해당 턴은 종료되야 jump나 back 후 해당 칸의 이벤트를 무시하는 것이 설명이 된된다고 생각해 변경했습니다.

**ClientHandler.java run 메소드 부분의 일부**{:data-align="center"}
```java
else {
    this.sum += diceNum;

    checkEnd(board);
    if (catchs(name, sum, dos)) {
        msg = "현재 " + this.sum + "칸 입니다.";
    } else {
        if (board[this.sum] == 1) {
            this.sum += diceNum;
            msg = "Jump!! " + diceNum + "칸을 점프해 현재 " + this.sum + "칸 입니다.";
        } else if (board[this.sum] == 2) {
            this.sum -= diceNum;
            msg = "Back!! " + diceNum + "칸을 후퇴해 현재 " + this.sum + "칸 입니다.";
        } else if (board[this.sum] == 3) {
            msg = "현재 " + this.sum + "칸 입니다.\n무인도에 걸려 한턴을 쉽니다.";
            rest = true;
        } else {
            msg = "현재 " + this.sum + "칸 입니다.";
        }
    }

    checkEnd(board);
}
dos.writeUTF(msg);
catchs(name, sum, dos);

private static boolean catchs(String name, int sum, DataOutputStream dos) throws IOException {
    for (ClientHandler handler : StreamServer.clients) {
        if (handler.name.equals(name))
            continue;
        if (sum > 0 && handler.sum > 0 && handler.sum == sum) {
            handler.sum = 0;
            dos.writeUTF("상대방의 말을 잡아 상대가 처음으로 돌아갑니다.");
            handler.dos.writeUTF(name + "에게 말이 잡혀 처음으로 돌아갑니다.");
            return true;
        }
    }
    return false;
}
```
catch 이벤트가 일어 났는지 아닌지를 확인하기 위해 해당 이벤트 메소드가 boolean 값을 return 하도록 했고, 그 값을 가지고 판단해 메시지를 보내도록 변경했습니다.

#### 무인도에 갇힌 후 잡히면 다시 원래대로 돌아와야 하는데 그러지 않음
**ClientHandler.java run 부분의 일부**{:data-align="center"}
```java
if (rest == true && board[sum] == 3) {
    msg = "다음 턴에 이동할 수 있습니다.";
    rest = false;
} else {
    this.sum += diceNum;
    if (rest)
        rest = false;

    // 아래 코드 생략
}
```
무인도에 갇히면 rest 값이 true가 되는데 잡혀도 rest가 true로 유지되는 문제가 있어서   
해당 말이 있는 칸을 조사한 후 해당 칸이 무인도 칸이면 원래대로 코드가 진행되고, 만약 아니라면 rest를 false로 변경할 수 있도록 했습니다.

#### 클라이언트가 0을 보내고 접속을 종료하는 것이 아닌 클라이언트가 보내는 조욜 신호를 잡고 서버가 접속 종료를 진행하게 변경
**ClientHandler.java run 부분의 일부**{:data-align="center"}
```java
if (diceNum == -1) {
    System.out.println(this.name + " is just leaved.");
    StreamServer.clients.remove(this);
    this.s.close();
    informLeave(this);
    break;
}
```

**StreamClient.java sendMessage 부분의 일부**{:data-align="center"}
```java
else if (game.equals("N") || game.equals("n")) {
    break;
}
```
원래 클라이언트가 0을 서버에 보내면 서버가 그 값을 받고 종료를 진행하게 계획했으나 클라이언트가 소켓 종료시 서버에 전달하는 값을 가지고 종료하도록 변경함.
1. 게임을 멈추겠다고 사용자가 n을 누르면 break가 실행되며 무한 루프가 깨집니다.
2. 아래에 socket.close가 동작하며 클라이언트를 종료하게 되고 그와 함께 서버에 -1값이 들어오게 됩니다.
3. 그럼 그 값을 서버가 잡고, 해당 클라이언트가 나간 것을 알게 되고, 벡터에서 해당 클라이언트를 제거하고, 소켓을 닫습니다.

### 참고
1. [스트림 서버 과제 제출 코드](https://github.com/Othkkartho/Board_Game_Server/tree/master)
2. [스트림 서버 과제 제출 이후 변경한 코드](https://github.com/Othkkartho/Board_Game_Server/tree/streamgame_improve)
3. [스트림 클라이언트 과제 제출 코드](https://github.com/Othkkartho/Board_Game_Client/tree/master)
4. [스트림 클라이언트 과제 제출 이후 변경한 코드](https://github.com/Othkkartho/Board_Game_Client/tree/streamgame_improve)