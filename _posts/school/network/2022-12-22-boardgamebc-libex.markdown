---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: network
title: 버퍼/채널 방식 보드게임 제작에 사용된 라이브러리 정리

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
date: 2022-12-22 20:00:00 +0900

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

대학교 네트워크보안 수업때 과제로 제출했던 스트림 방식과 버퍼-채널 방식의 보드 게임 제작을 진행하였는데 해당 포스트에서는 버퍼/채널 방식으로 제작한 게임의 라이브러리를 설명드리겠습니다.

<!-- outline-end -->

### 버퍼/채널 기반 게임 제작에 사용된 라이브러리의 자세한 설명
**Buffer_Channel_Server.java**{:data-align="center"}
```java
import java.io.IOException;
import java.net.InetSocketAddress;              // 1
import java.nio.channels.SelectionKey;          // 2
import java.nio.channels.Selector;              // 3
import java.nio.channels.ServerSocketChannel;   // 4
import java.nio.channels.SocketChannel;         // 5
import java.util.*;
import java.util.concurrent.TimeUnit;           // 6
```
1. 또한 바인딩, 연결 또는 반환 값으로 소켓에서 사용하는 불변 개체를 제공합니다. 해당 과제에서는 서버를 열기 위해 사용되었습니다.
2. 추상 클래스로서 Selector로 SelectableChannel의 등록을 나타내는 토큰입니다.
    - SelectableChannel은 다중화된 Non-Blocking I/O 작업을 위한 Selector를 정의하는 추상 클래스입니다.
    - 해당 클래스는 셀렉터에 채널을 등록할 때마다 선택 키가 생성되는 역할을 위해 사용되었습니다. 해당 키는 취소 메서드 호출이나 채널, Selector를 받을 때 까지 유효한 상태로 유지됩니다.
3. SelectableChannel 개체의 멀티플렉서 라이브러리입니다. Selector는 시스템의 기본 Selector provider를 사용해 새로운 Selector를 만듭니다.
    - 멀티플렉서란 Stream에서 사용한 방식처럼 멀티스레드 기반이 아닌 입출력 대상을 묶어 관리하는 방식의 서비스입니다. 즉 다수의 프로세스를 생성하는 방식으로 서비스를 제공하는 것을 말합니다.
    - 본 프로젝트에서도 Non-Blocking 방식의 멀티플렉서 서비스의 서버와 클라이언트를 만들기 위해 사용했습니다.
4. 스트림 지향의 Listening 소켓입니다.
    - 인터넷 프로토콜 소켓에 대한 서버 소켓 채널을 열기 위해 사용했습니다.
5. SocketChannel은 스트림 지향 연결 소켓입니다.
    - 서버에 연결한 클라이언트의 정보를 저장하고, 클라이언트의 경우 서버와의 연결과 서버 정보 저장을 위해 사용했습니다.
6. 일정 시간이 지난 후 동작을 시키려고 할 때 사용하는 라이브러리입니다.

**ClientHandler.java**{:data-align="center"}
```java
import java.io.IOException;
import java.nio.channels.SocketChannel;
import java.util.Random;
import java.util.concurrent.TimeUnit;
```

**HelperMethods.java**{:data-align="center"}
```java
import java.io.IOException;
import java.nio.ByteBuffer;             // 1
import java.nio.channels.SocketChannel;
```
1. 바이트 버퍼는 바이트를 담는 버퍼로써 서버와 클라이언트 간의 통신을 위한 데이터 전송을 위해 사용되었습니다.

**Buffer_Channel_Client.java**{:data-align="center"}
```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.security.SecureRandom;
import java.util.*;
```

**PrintIt.java**{:data-align="center"}
```java
import java.nio.channels.SocketChannel;
import java.util.StringTokenizer;       // 1
```
1. 문자열을 특정 문자로 나눌 수 있습니다.
    - 전체적인 데이터 처리를 위해 사용하였습니다.

#### Selector의 open 동작 설명
**Buffer_Channel_Server.java acceptClient 메소드**{:data-align="center"}
```java
selector = Selector.open();
```
Selector를 엽니다.


**Selector.java**{:data-align="center"}
```java
public static Selector open() throws IOException {
    return SelectorProvider.provider().openSelector();
}
```
시스템 전반의 기본 SelectorProvider 객체의 openSelector 메서드를 호출해 생성하고, 해당 Selector 객체를 return합니다.

#### ServerSocketChannel의 open 동작 설명
**Buffer_Channel_Server.java acceptClient 메소드**{:data-align="center"}
```java
ServerSocketChannel sschannel = ServerSocketChannel.open();
```
서버 생성을 위한 변수를 만들기 위해 open 메서드를 호출합니다.

**Selector.java**{:data-align="center"}
```java
public static ServerSocketChannel open() throws IOException {
    return SelectorProvider.provider().openServerSocketChannel();
}
```
인터넷 프로토콜 소켓에 대한 서버 소켓 채널을 엽니다.   
새 채널은 시스템 전반의 기본 SelectorProvider 객체의 openServerSocketChannel 메서드를 호출해 생성하고, 해당 객체를 return합니다.

#### InetSocketAddress의 동작 설명
**Buffer_Channel_Server.java acceptClient 메소드**{:data-align="center"}
```java
sschannel.bind(new InetSocketAddress(5001));
```
서버를 열기 위해 서버가 가져야 하는 정보를 서버 소켓 채널에 저장하려고 합니다.

**Selector.java**{:data-align="center"}
```java
public InetSocketAddress(int port) {
    this(InetAddress.anyLocalAddress(), port);
}
```
IP 주소는 와일드카드 주소이고 포트 번호는 입력된 값으로 지정한 소켓 주소를 만듭니다.   
실행 결과 값은 *0.0.0.0/0.0.0.0:5001* 입니다.    
포트만이 아닌 IP 주소를 직접 입력해 사용할 수 있는 메서드도 있습니다.

#### ServerSocketChannel의 기타 동작 설명
**Buffer_Channel_Server.java acceptClient 메소드**{:data-align="center"}
```java
sschannel.bind(new InetSocketAddress(5001));
```
InetSocketAddress가 실행되어 소켓 주소를 만들었다면 해당 주소를 입력으로 받아 다음 동작을 진행합니다.

**Selector.java**{:data-align="center"}
```java
public final ServerSocketChannel bind(SocketAddress local) throws IOException {
    return bind(local, 0);
}
```
채널의 소켓을 로컬 주소에 바인드하고 연결을 수신하도록 소켓을 구성합니다.
만약 보류중인 최대 연결 수인 backlog를 설정하지 않는다면 위와 같은 메서드가 호출되지만 설정할 수 있는 메서드가 ServerSocketChannelInpl에 구현되어 있습니다.    
그러면 [/[0:0:0:0:0:0:0:0]:5001] 과 같은 ServerSocketChannelImpl 형태의 값이 생성되어 sschannel 변수에 저장됩니다.   \n

이렇게 서버 채널 생성이 완료되면 Selector를 사용하기 위해 아래의 코드를 추가합니다.
**Buffer_Channel_Server.java acceptClient 메소드**{:data-align="center"}
```java
sschannel.configureBlocking(false);
```

**Selector.java**{:data-align="center"}
```java
public final SelectableChannel configureBlocking(boolean block) throws IOException {
    synchronized (regLock) {
        if (!isOpen())
            throw new ClosedChannelException();
        boolean blocking = !nonBlocking;
        if (block != blocking) {
            if (block && haveValidKeys())
                throw new IllegalBlockingModeException();
            implConfigureBlocking(block);
            nonBlocking = !block;
        }
    }
    return this;
}
```
해당 메소드는 Blocking 메서드들이 Non-Blocking으로 동작하게 하기 위해 설정을 확인하고 해당 동작들을 진행합니다.   
그럼 서버소켓채널의 메서드인 sschannel의 **nonBlocking** 값이 **false**에서 **true**로 변경되는 것을 확인하실 수 있습니다.

**Buffer_Channel_Server.java acceptClient 메소드**{:data-align="center"}
```java
sschannel.register(selector, SelectionKey.OP_ACCEPT);
```
서버소켓채널에 selector를 등록시킵니다.

**Selector.java**{:data-align="center"}
```java
public final SelectionKey register(Selector sel, int ops)
    throws ClosedChannelException
{
    return register(sel, ops, null);
}
```
셀렉터에 채널을 등록하고 SelectionKey를 반환하는 작업을 진행합니다.   
변수인 ops는 등록을 원하는 키의 값입니다. OP-AACEPT의 경우 `public static final int OP_ACCEPT = 1 << 4;`로 정의되어 있기 때문에 비트를 4번 왼쪽으로 이동시키고 빈 공간을 0으로 채워 10 000(2) 16(10) 이 입력되었습니다.   \n

해당 코드의 모든 진행이 완료되면 sschannel의 keys 변수에 SelectionKeyImpl 형식으로 채널과 selector가 저장되어 있고, 위에서 언급했던 Ops또한 저장되어 있습니다. 이는 selector의 keys에도 동일한 값이 저장되어 있음을 진행해 보시면 확인 하실 수 있습니다.

#### Selector의 select 동작 설명
**Buffer_Channel_Server.java acceptClient 메소드**{:data-align="center"}
```java
selector.select();
```
해당 메서드는 Blocking 형태로 동작합니다. selector에 등록된 채널에 등록된 SelectionKey 동작이 요청되었는지를 확인하고 확인이 되면 해당 채널이 I/O 작업을 위해 준비된 키 세트를 선택합니다.

**Buffer_Channel_Server.java acceptClient 메소드**{:data-align="center"}
```java
Set<SelectionKey> keys = selector.selectedKeys();
```
selector.selectedKeys를 통해 selector에 등록되있는 Key값을 가져와 데이터를 중복해서 저장할 수 없는 Set 형식으로 저장합니다.

**Buffer_Channel_Server.java acceptClient 메소드**{:data-align="center"}
```java
Iterator<SelectionKey> iterator = keys.iterator();
```
저장한 값을 다시 검색을 위해 원소들을 탐색하는 포인터와 같은 역할을 하는 순환자(Iterator) 형식으로 만들어 저장합니다.

**Buffer_Channel_Server.java acceptClient 메소드**{:data-align="center"}
```java
if (key.isAcceptable()) {
    SocketChannel client = sschannel.accept();
}
```
서버가 연결 요청을 한다면 그 요청을 받습니다. 즉 요청이 오기를 기다리다가 요청을 받았던 Stream 방식과는 다르게 요청이 오면 그 요청을 받는 메서드를 실행시키는 방식입니다.

#### SocketChannel read write 동작 설명
**HelperMethods.java receiveMessage 메서드의 코드 중 일부**{:data-align="center"}
```java
socketChannel.read(byteBuffer)
```

**SocketChannelImpl.java**{:data-align="center"}
```java
public int read(ByteBuffer buf) throws IOException {
Objects.requireNonNull(buf);

readLock.lock();
try {
    ensureOpenAndConnected();
    boolean blocking = isBlocking();
    int n = 0;
    try {
        beginRead(blocking);

        // check if connection has been reset
        if (connectionReset)
            throwConnectionReset();

        // check if input is shutdown
        if (isInputClosed)
            return IOStatus.EOF;

        configureSocketNonBlockingIfVirtualThread();
        n = IOUtil.read(fd, buf, -1, nd);       // 1
        if (blocking) {
            while (IOStatus.okayToRetry(n) && isOpen()) {
                park(Net.POLLIN);
                n = IOUtil.read(fd, buf, -1, nd);
            }
        }
    } catch (ConnectionResetException e) {
        connectionReset = true;
        throwConnectionReset();
    } finally {
        endRead(blocking, n > 0);
        if (n <= 0 && isInputClosed)
            return IOStatus.EOF;
    }
    return IOStatus.normalize(n);               // 2
} finally {
    readLock.unlock();
}
```
1. 해당 위치에서 실질적인 읽기 작업이 일어납니다. IOUtil에서 실제 작업을 진행하는군요.   
    - 읽기가 완료 되면 buf에 해당 채널에서 보낸 값이 입력됩니다.
2. 모든 작업을 마치면 바이트 버퍼에는 입력받은 값이 저장되게 됩니다.
그리곤 다시 해당 메소드로 돌아가 전달받은 바이트 데이터를 string으로 변환시켜 통신을 진행합니다.

**HelperMethods.java sendMessage 메서드의 코드 중 일부**{:data-align="center"}
```java
socketChannel.write(buffer);
```
String 타입의 값의 데이터를 byte로 변환해 해당 채널에 보내기 위한 메서드 호출입니다.

**SocketChannelImpl.java**{:data-align="center"}
```java
@Override
public int write(ByteBuffer buf) throws IOException {
    Objects.requireNonNull(buf);
    writeLock.lock();
    try {
        ensureOpenAndConnected();
        boolean blocking = isBlocking();
        int n = 0;
        try {
            beginWrite(blocking);
            configureSocketNonBlockingIfVirtualThread();
            n = IOUtil.write(fd, buf, -1, nd);  // 1
            if (blocking) {
                while (IOStatus.okayToRetry(n) && isOpen()) {
                    park(Net.POLLOUT);
                    n = IOUtil.write(fd, buf, -1, nd);
                }
            }
        } finally {
            endWrite(blocking, n > 0);
            if (n <= 0 && isOutputClosed)
                throw new AsynchronousCloseException();
        }
        return IOStatus.normalize(n);
    } finally {
        writeLock.unlock();
    }
}
```
1. write 동작 또한 IOUtil.write에 의해 write 동작이 일어나는 것을 확인할 수 있습니다.

#### StringTokenizer의 동작 설명
**Buffer_Channel_Server.java acceptClient의 일부**{:data-align="center"}
```java
StringTokenizer tokenizer = new StringTokenizer(received, "#"); // 1
String what = tokenizer.nextToken();                            // 2
String data = tokenizer.nextToken();                            // 2
```
1. 해당 문자(received)를 토큰화 한다고 기준으로 잡아 놓은 문자(#)을 기준으로 나누고, 그 값을 tokenizer로 등록합니다.
2. 등록된 값을 나누어 저장합니다. 2개 이상으로도 토큰화가 가능합니다.

**StringTokenizer.java**{:data-align="center"}
```java
public StringTokenizer(String str, String delim) {      // 1
    this(str, delim, false);
}

public StringTokenizer(String str, String delim, boolean returnDelims) {    // 2
    currentPosition = 0;
    newPosition = -1;
    delimsChanged = false;
    this.str = str;
    maxPosition = str.length();
    delimiters = delim;
    retDelims = returnDelims;
    setMaxDelimCodePoint();
}

private void setMaxDelimCodePoint() {       // 3
    if (delimiters == null) {
        maxDelimCodePoint = 0;
        return;
    }

    int m = 0;
    int c;
    int count = 0;
    for (int i = 0; i < delimiters.length(); i += Character.charCount(c)) {
        c = delimiters.charAt(i);
        if (c >= Character.MIN_HIGH_SURROGATE && c <= Character.MAX_LOW_SURROGATE) {
            c = delimiters.codePointAt(i);
            hasSurrogates = true;
        }
        if (m < c)
            m = c;
        count++;
    }
    maxDelimCodePoint = m;

    if (hasSurrogates) {
        delimiterCodePoints = new int[count];
        for (int i = 0, j = 0; i < count; i++, j += Character.charCount(c)) {
            c = delimiters.codePointAt(j);
            delimiterCodePoints[i] = c;
        }
    }
}
```
1. 토큰화 하기 위해 메서드를 호출합니다.
2. 전달받은 데이터들로 해당 클래스의 변수들을 초기화 시킵니다.
3. 실질적인 토큰화를 실시합니다.
