---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: network
title: 스트림 방식의 보드게임 제작에 사용된 라이브러리 설명

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
date: 2022-12-20 22:00:00 +0900

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

대학교 네트워크보안 수업때 과제로 제출했던 스트림 방식과 버퍼-채널 방식의 보드 게임 제작을 진행하였는데 해당 포스트에서는 스트림 방식의 게임 제작에 사용된 라이브러리들을 설명합니다.

<!-- outline-end -->
### Stream 기반 게임 제작에 사용된 라이브러리의 자세한 설명
```java:StreamServer.java
import java.io.DataInputStream; // 1
import java.io.DataOutputStream; // 2
import java.io.IOException; // 3
import java.net.ServerSocket; // 4
import java.net.Socket; // 5
import java.util.Vector; // 6
```
1. 해당 라이브러리는 DataInput 인터페이스의 구현체이며 FilterInputStream을 상속받습니다.
    - DataInput은 이진 스트림에서 바이트를 읽고 그 데이터를 자바의 기본적인 유형으로 데이터를 재구성할 수 있도록 하는 인터페이스입니다. 또한 UTF-8 문자열을 재구성 할 수 있는 기능도 제공합니다.
    - FilterInputStream은 InputStream 추상 클래스를 상속받는 라이브러리로 모든 요청을 포장된 입력 스트림에 전달하는 InputStream의 메서드를 재정의합니다.
        - InputStream은 바이트 입력 스트림을 나타내는 모든 클래스의 부모 클래스입니다.
    - DataInputStream은 프로그램이 기본적인 입력 스트림에서 기본적인 자바의 데이터 유형을 기계 독립적인 방식으로 읽을 수 있게 합니다. 해당 라이브러리는 데이터 출력 스트림과 상호작용합니다. 해당 라이브러리는 멀티 스레드에서 사용하기에 적합하지 않습니다. 만약 사용하려면 입력 스트림에 대한 액세스를 동기화로 제어해야 합니다.
2. DataOutputStream은 DataOutput 인터페이스의 구현체이며 FilterOutputStream을 상속받습니다.
    - DataOutput, FilterOutputStream, DataOutputStream은 위의 DataInputStream과 반대의 역할을 합니다.
3. IOException은 입출력과 관련한 예외가 발생했을 때를 의미합니다. 주로 예외처리를 위해 많이 사용합니다.
4. 서버 소켓을 구현합니다. 서버 소켓은 네트워크를 통해 요청이 들어올 때까지 Blocking 메서드로 동작하고, 요청이 발생하면 일부 작업을 수행 후 다음 요청에 대해 결과를 반환할 수 있습니다. 실제 작업은 SoketImp 클래스의 인스턴스에 의해 수행됩니다.
5. Socket은 클라이언트 소켓을 구현합니다. 실제 작업은 SocketImp 클래스의 인스턴스에 의해 수행됩니다.
6. ArrayList와 동일한 구조를 가지고있으며, 배열의 크기가 변함에 따라 자동으로 크기가 조절되며, 동기화라는 특징이 있어 여러 클라이언트를 스레드로 실행해야 하는 해당 게임에 사용했습니다.

```java:StreamClient.java
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.InetAddress;    // 1
import java.net.Socket;
import java.util.Random;
import java.util.Scanner;
```
- IP 주소를 확인하는 라이브러리입니다.

#### Serversocket의 동작 설명
```java:StreamServer.java
ServerSocket server = new ServerSocket(3005);
```
서버가 시작되면 서버를 열 포트를 지정하고, 초기화를 진행합니다.

```java:ServerSocket.java
public ServerSocket(int port) throws IOException {  // 1
    this(port, 50, null);
}

public ServerSocket(int port, int backlog, InetAddress bindAddr) throws IOException {   // 2
    if (port < 0 || port > 0xFFFF)
        throw new IllegalArgumentException("Port value out of range: " + port);
    if (backlog < 1)
        backlog = 50;

    this.impl = createImpl();   // 2
    try {
        bind(new InetSocketAddress(bindAddr, port), backlog);   // 4
    } catch (IOException | SecurityException e) {
        close();
        throw e;
    }
}

private static SocketImpl createImpl() {    // 3
    SocketImplFactory factory = ServerSocket.factory;
    if (factory != null) {
        return factory.createSocketImpl();
    } else {
        return SocketImpl.createPlatformSocketImpl(true);
    }
}
```
1. 포트를 입력으로 받으면 포트와 수신 대기열의 최대 길이를 기본으로 입력하고, 서버가 바인딩할 로컬 주소를 입력으로 줍니다.
2. 전달받은 데이터를 가지고 서버 소켓을 만드는 createImpl을 호출합니다. 그럼 기본적인 소켓이 만들어져 변수에 저장됩니다.
3. 서버 소켓을 만들 SocketImpl를 만듭니다.
4. 서버 소켓을 특정 주소 (IP 주소 및 포트 번호)에 바인딩합니다.

```java:InetSocketAddress.java
public InetSocketAddress(InetAddress addr, int port) {
    holder = new InetSocketAddressHolder(
                    null,
                    addr == null ? InetAddress.anyLocalAddress() : addr,
                    checkPort(port));
}
```
IP 주소와 포트를 전달받아 소켓 주소를 만듭니다.   \n

위와 같은 작업이 끝나면 서버가 사용자가 지정한 포트로 열리게됩니다.

#### Socket의 동작 설명
```java:StreamServer.java
Socket socket = server.accept();
```
서버가 클라이언트의 접속을 block 상태로 기다립니다.

```java:ServerSocket.java
public Socket accept() throws IOException {
    if (isClosed())
        throw new SocketException("Socket is closed");
    if (!isBound())
        throw new SocketException("Socket is not bound yet");
    Socket s = new Socket((SocketImpl) null);   // 1
    implAccept(s);      // 2
    return s;
}

protected Socket(SocketImpl impl) throws SocketException {
    checkPermission(impl);
    this.impl = impl;
}

private static Void checkPermission(SocketImpl impl) {
    if (impl == null) {
        return null;
    }
    @SuppressWarnings("removal")
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        sm.checkPermission(SecurityConstants.SET_SOCKETIMPL_PERMISSION);
    }
    return null;
}
```
연결을 수신하고 수락하는 메소드입니다.   
새로운 null 값을 가진 소켓을 하나 만들고, 그 소켓을 저장합니다.   
그리고 주석 2번의 메소드를 호출합니다. 해당 메소드는 아래에 작성되어 있습니다.

```java:ServerSocket.java
protected final void implAccept(Socket s) throws IOException {  // 1
    SocketImpl si = s.impl;

    if (si == null) {
        si = implAccept();  // 2
        s.setImpl(si);      // 5
        s.postAccept();     // 6
        return;             // 7
    }

    if (si instanceof DelegatingSocketImpl) {
        si = ((DelegatingSocketImpl) si).delegate();
        assert si instanceof PlatformSocketImpl;
    }

    ensureCompatible(si);
    if (impl instanceof PlatformSocketImpl) {
        SocketImpl psi = platformImplAccept();
        si.copyOptionsTo(psi);
        s.setImpl(psi);
        si.closeQuietly();
    } else {
        s.impl = null;
        try {
            customImplAccept(si);
        } finally {
            s.impl = si;
        }
    }
    s.postAccept();
}

private void implAccept(SocketImpl si) throws IOException { // 3
    assert !(si instanceof DelegatingSocketImpl);

    try {
        impl.accept(si);    // 4
    } catch (SocketTimeoutException e) {
        throw e;
    } catch (InterruptedIOException e) {
        Thread thread = Thread.currentThread();
        if (thread.isVirtual() && thread.isInterrupted()) {
            close();
            throw new SocketException("Closed by interrupt");
        }
        throw e;
    }

    @SuppressWarnings("removal")
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        try {
            sm.checkAccept(si.getInetAddress().getHostAddress(), si.getPort());
        } catch (SecurityException se) {
            si.close();
            throw se;
        }
    }
}
```
1. 서버 소켓의 하위 클래스는 이 메소드를 사용해 소켓의 하위 클래스를 반환하기 위해 accept를 재정의합니다.
2. 새로운 SocketImpl과의 연결을 허용하기 위한 메소드 호출입니다.
3. 지정된 SocketImpl가 피어에 연결되도록 새 연결을 수락합니다.
4. 수락을 위해 block 됩니다. 그리고 클라이언트가 접속을 시도하면 si 변수에 접속을 요청한 클라이언트의 소켓 정보가 저장되고 그 값들이 return되어 주석 2번의 si 변수에 저장되게 됩니다.
5. Socket 라이브러리에 impl 변수에도 해당 정보를 저장합니다.
6. accept 호출 후 connected, created, bound 플레그를 true로 설정합니다.
7. 그렇게 최종적으로 접속한 클라이언트의 소켓이 StreamServer의 socket 변수에 저장됩니다.

#### DataInputStream의 동작 설명
```java:StreamServer.java
DataInputStream dis = new DataInputStream(socket.getInputStream());
```
DataInputStream을 클라이언트 소켓의 InputStream을 가져와 초기화를 진행합니다.

```java:DataInputStream.java
public DataInputStream(InputStream in) {
    super(in);
}
```
지정된 기본 InputStream을 사용하는 DataInputStream을 만듭니다.

```java:FilterInputStream.java
protected FilterInputStream(InputStream in) {   // 1
    this.in = in;
}

private byte bytearr[] = new byte[80];  // 2
private char chararr[] = new char[80];  // 2

private byte readBuffer[] = new byte[8]; // 2
```
1. 나중에 사용할 수 있도록 this.in 필드에 인수를 할당해 FilterInputStream을 만듭니다.
2. 읽기를 위해 초기화 작업을 진행합니다.

- - -

```java:StreamServer.java
String name = dis.readUTF();
```
클라이언트에게 값을 전달받기 위해 readUTF 메소드를 호출합니다.

```java:DataInputStream.java
public final String readUTF() throws IOException {
    return readUTF(this);
}
```
호출된 메소드는 실제 읽는 작업을 진행할 readUTF를 호출하고 해당 메소드에서 클라이언트가 write 작업을 진행할 때까지 Block 상태로 대기합니다.

```java:DataInputStream.java
public static final String readUTF(DataInput in) throws IOException {
    int utflen = in.readUnsignedShort();        // 1
    byte[] bytearr = null;
    char[] chararr = null;
    if (in instanceof DataInputStream dis) {    // 2
        if (dis.bytearr.length < utflen){
            dis.bytearr = new byte[utflen*2];
            dis.chararr = new char[utflen*2];
        }
        chararr = dis.chararr;
        bytearr = dis.bytearr;
    } else {
        bytearr = new byte[utflen];
        chararr = new char[utflen];
    }

    int c, char2, char3;
    int count = 0;
    int chararr_count=0;

    in.readFully(bytearr, 0, utflen);

    while (count < utflen) {                // 3
        c = (int) bytearr[count] & 0xff;
        if (c > 127) break;
        count++;
        chararr[chararr_count++]=(char)c;
    }

    while (count < utflen) {                // 5
        c = (int) bytearr[count] & 0xff;
        switch (c >> 4) {
            case 0, 1, 2, 3, 4, 5, 6, 7 -> {
                count++;
                chararr[chararr_count++]=(char)c;
            }
            case 12, 13 -> {
                count += 2;
                if (count > utflen)
                    throw new UTFDataFormatException(
                        "malformed input: partial character at end");
                char2 = (int) bytearr[count-1];
                if ((char2 & 0xC0) != 0x80)
                    throw new UTFDataFormatException(
                        "malformed input around byte " + count);
                chararr[chararr_count++]=(char)(((c & 0x1F) << 6) |
                                                (char2 & 0x3F));
            }
            case 14 -> {
                count += 3;
                if (count > utflen)
                    throw new UTFDataFormatException(
                        "malformed input: partial character at end");
                char2 = (int) bytearr[count-2];
                char3 = (int) bytearr[count-1];
                if (((char2 & 0xC0) != 0x80) || ((char3 & 0xC0) != 0x80))
                    throw new UTFDataFormatException(
                        "malformed input around byte " + (count-1));
                chararr[chararr_count++]=(char)(((c     & 0x0F) << 12) |
                                                ((char2 & 0x3F) << 6)  |
                                                ((char3 & 0x3F) << 0));
            }
            default ->
                throw new UTFDataFormatException(
                    "malformed input around byte " + count);
        }
    }
    return new String(chararr, 0, chararr_count);
}
```
1. 값을 전달받을 때 까지 대기하다가 값이 도착하면 그 값의 길이를 저장합니다.
2. DataInputStream(dis)이 DataInput을 상독 받았다면 해당 조건문이 동작합니다.
    - 전달받은 값이 UTF-8인지 ASCII 코드표인지를 확인해 해당 값에 맞는 배열 길이로 초기화를 시작합니다.
3. 전달받은 바이트를 int로 변환한 결과 그 값이 ASCII 코드로 작성되었다면 그 값을 char 배열에 저장합니다.
4. 전달받은 바이드를 int로 변환한 결과 그 값이 UTF-8로 작성되었다면 그 값을 char 배열에 저장합니다.


#### DataOutputStream의 동작 설명
초기화 하는 동작 방식은 Input과 거의 흡사하기 때문에 따로 넣지는 않았습니다.

```java:StreamClient.java
dos.writeUTF(name);
```
값을 넣으면 해당 메소드가 호출됩니다.

```java:DataOutputStream.java
public final void writeUTF(String str) throws IOException {
    writeUTF(str, this);
}
```
해당 메소드는 전달할 값과, 초기화 시에 저장해 놓은 DataOutputStream을 입력으로 실제 해당 작업을 할 메소드를 호출합니다.

```java:DataOutputStream.java
static int writeUTF(String str, DataOutput out) throws IOException {
    final int strlen = str.length();    // 1
    int utflen = strlen;

    for (int i = 0; i < strlen; i++) {  // 2
        int c = str.charAt(i);
        if (c >= 0x80 || c == 0)
            utflen += (c >= 0x800) ? 2 : 1;
    }

    if (utflen > 65535 || utflen < strlen)
        throw new UTFDataFormatException(tooLongMsg(str, utflen));

    final byte[] bytearr;
    if (out instanceof DataOutputStream dos) {  // 3
        if (dos.bytearr == null || (dos.bytearr.length < (utflen + 2)))
            dos.bytearr = new byte[(utflen*2) + 2];
        bytearr = dos.bytearr;
    } else {
        bytearr = new byte[utflen + 2];
    }

    int count = 0;
    bytearr[count++] = (byte) ((utflen >>> 8) & 0xFF);
    bytearr[count++] = (byte) ((utflen >>> 0) & 0xFF);

    int i = 0;
    for (i = 0; i < strlen; i++) {  // 4
        int c = str.charAt(i);
        if (c >= 0x80 || c == 0) break;
        bytearr[count++] = (byte) c;
    }

    for (; i < strlen; i++) {   // 5
        int c = str.charAt(i);
        if (c < 0x80 && c != 0) {
            bytearr[count++] = (byte) c;
        } else if (c >= 0x800) {
            bytearr[count++] = (byte) (0xE0 | ((c >> 12) & 0x0F));
            bytearr[count++] = (byte) (0x80 | ((c >>  6) & 0x3F));
            bytearr[count++] = (byte) (0x80 | ((c >>  0) & 0x3F));
        } else {
            bytearr[count++] = (byte) (0xC0 | ((c >>  6) & 0x1F));
            bytearr[count++] = (byte) (0x80 | ((c >>  0) & 0x3F));
        }
    }
    out.write(bytearr, 0, utflen + 2);      // 6
    return utflen + 2;
}
```
1. 다음 반복문을 위해 미리 전달받은 값의 길이를 확인합니다.
2. ASCII 문자열의 경우 1글자당 1byte의 길이를 가지지만 UTF-8의 경우 2byte를 가지기 때문에 이를 대비해 UTF-8 문자열이 있는지 확인 후 utflen의 길이를 늘려 최적화 시킵니다.
3. DataOutputStream(dos)이 DataOutput(out)을 상속받았다면 dos의 bytearr가 null인지 확인 후 bytelist를 생성합니다.
4. ASCII 코드에 맞춘 반복문입니다. 만약 입력 값이 아스키 코드라면 해당 값을 byte로 변환해 bytearr 변수에 저장하지만 UTF-8이라면 건너 뜁니다.
5. UTF-8로 작성된 문자열을 바이트 형식으로 저장하는 반복문입니다.
6. 해당 작업이 모두 완료되면 DataOutput에 값을 저장하고, 해당 값을 보냅니다.
