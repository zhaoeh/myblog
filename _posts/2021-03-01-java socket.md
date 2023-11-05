---
layout:     post
title:      Java网络编程
subtitle:   Socket是java网络编程的基础
categories: [Java中级知识]
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 1. Socket原理
1.  网络编程主要依赖于Jdk中的Socket类和ServerSocket类。Socket类充当客户端，ServerSocket类充当服务器端。  
2.  Socket将指定内容通过输出流OutputStream输出到服务器端，服务器端的Socket对象会通过输出流InputStream接收Socket发送的数据，并对该数据进行处理，处理完毕后可能会通过输出流输出到Socket客户端的输入流中进行响应。  
3.  Socket编程中注意客户端启动的Socket不要定义为成员变量，一旦定义为成员变量，将会面临单利模式的多线程安全风险，因为成员变量有可能是共享可变成员，极有可能被多个线程对象共享，如此一来，在Socket客户端和Socket服务端进行网络通信遇到较大的传输报文时，有可能造成IO阻塞，导致成员变量的Socket对象被回收，此时服务端误以为客户端的Socket已经失效就主动关闭Socket链接，这样导致客户端总是报错Connection reset。  

# 2. 网络通信协议 TCP 和 UDP
## 2.1 TCP/IP协议
1.  tcp/ip协议是实现任何远程通信的基础协议。只要牵扯远程通信，其底层都是tcp/ip协议。  
2.  java中的Socket就是对tcp/ip协议的包装，可以方便的实现远程通信。  
3.  任何作为服务方的ServerSocket或者类似的服务器比如Servlet容器，其本身就是多线程方式启动的，至于线程的启动方式可能是单例方式启动也可能是多例方式启动。采取多线程方式启动socket服务端，目的是为了同时处理多个socket客户端的并发请求。  
4.  tcp请求本身分为同步和异步，默认tcp请求都是同步请求：  
    同步：只要Socket和ServerSocket建立连接并存在数据交互，则客户端线程就必须等待服务器端将数据处理完毕后才能继续执行。  
    异步：客户端和服务端只建立Socket连接而不存在任何的数据交互，则客户端实际上并依赖服务端的响应，所以只要客户端和服务端建立完连接后，客户端线程即立即返回做自己的事情。  
5.  扩展：现实中，tcp本身的异步方式几乎每用武之地。因为很少存在客户端只和服务端建立连接而不存在任何数据交互的场景。  
    因此，对于既需要获取服务方的响应数据又想实现异步通信，只能依靠代码自己控制。  
    比如客户端通过多线程去实现异步、比如http请求依赖ajax的异步请求实现的局部刷新等。  

## 2.2 UDP协议


## 2.3 HTTP协议
1.  对于http而言，虽然底层依赖的是tcp协议，但是Http和客户端浏览器有关。  
2.  http建立连接和java中的同步是一样的。  
3.  http建立的链接默认是同步的，不管是否需要服务端的响应结果，其默认就是同步的。  
4.  http链接如果需要异步需要使用ajax技术或者使用多线程创建httpClient。  

# 3. 连接超时时间
Socket客户端和ServerSocket服务端通过Socket完成三次握手建立tcp连接，设置一个时间值，超过这个时间如果连接还没有成功建立，则返回连接超时。  

# 4. 读写超时时间
Socket客户端和ServerSocket服务端通过Socket完成三次握手建立tcp连接，开始传输数据，读写输入流和输出流也需要时间限制，如果超过指定时间则报读写超时。  
一般来说，连接超时时间设置的比较小，而读写超时时间设置值比较大一些。  
这两者没有任何关系，读写超时时间是在客户端和服务端之间已经开始传递数据后才计算的，此时tcp链接已经建立完成了。  

# 5. 从rpc的幂等性探讨读写超时、重试机制的底层
## 5.1 rpc遵守tcp/ip协议
不论是http还是rpc等远程通信协议，其底层都遵守tcp/ip协议。  

## 5.2 tcp/ip协议的同步和异步
任何通信协议几乎都离不开tcp/ip协议的支持，tcp/ip协议本身分为同步和异步：  
1.  Request-发出后，客户端不需要等待服务端的响应流，属于tcp/ip异步通信。  
2.  Request-Response，必须满足“客户端-服务端”的通信模式，即客户端发出request后，必须同步等待服务端的响应数据，属于tcp/ip同步通信。  
注意：上述同步和异步是tcp/ip协议本身支持的同步和异步方式，意味着在不使用多线程等方式人为干涉tcp/ip协议的方式下，其本身就支持的同步和异步方式。  

## 5.3 客户端和服务端的读写超时前提
Socket要想让读写超时，前提必须得有读有写，即：客户端需要得到服务端响应流数据的前提下，才会存在读写超时的可能性。  
意味着，读写超时的前提是tcp/ip本身的同步方式。如果客户端不需要得到服务端响应数据，则此时通信方式为异步，客户端线程建立完链接后便立即返回，而永不可能出现什么所谓读写超时。  

## 5.4 rpc远程调用中的超时重试机制
1.  rpc框架底层都是基于tcp/ip进行远程通信的。  
2.  当没有使用分布式架构前，所有应用共享同一个JVM，此时不管是同步调用还是异步调用，不管是同步回调还是异步回调，都不牵扯网络超时问题，因为同属一个JVM，不涉及远程网络通信。  
3.  一旦使用rpc，比如dubbo，那么consumer和provider之间的远程通信就和通信超时有关系了，此时超时机制和重试机制取决于：  
    consumer发起的请求是否需要服务方返回数据，即服务方接口是否是具有返回值的方法。  
    如果服务方的接口是没有返回值的方法，那么超时和重试机制将不会影响客户端的调用，因为客户端压根就不需要等待服务端的响应结果，服务端就慢慢执行就行了。  
4.  最终结论：tcp通信（包括一切基于tcp协议的通信）和rpc远程调用中，超时机制和重试机制是否有效，取决于客户端是够需要等待服务端的响应数据。  
    如果服务方不需要返回数据，则客户端不会超时，哪怕服务器执行100年；  
    如果客户端需要等待服务方响应数据，则客户端有可能超时，因为此时客户端会一直阻塞去等待服务端数据的响应。  

# 6. socket编程案例
## 6.1 TCP服务端
zeh.myjavase.code28socket.demo01.TCPServerSocket
```java
package zeh.myjavase.code28socket.demo01;

import java.io.IOException;
import java.io.PrintStream;
import java.net.ServerSocket;
import java.net.Socket;

// ServerSocket类：TCP服务器端类
public class TCPServerSocket {
    public static void main(String args[]) throws IOException {
        ServerSocket server = null;
        Socket client = null;
        PrintStream out = null;
        server = new ServerSocket(9999);

        System.out.println("服务端运行，等待客户端连接...");
        // 注意：服务端现在启动运行的只有一个主线程，主线程在accept()方法上等待客户端请求的到来。
        // accept()方法用来获取客户端Socket对象，获取到之后就能够拿到客户端请求。
        // 如果服务端只启动主线程去处理客户端上送的请求，一旦这个处理动作是重量级的，那么主线程的执行将非常缓慢。
        // 服务端获取到客户端Socket后，需要通过该socket对象去读写IO流，以便从客户端请求中获取数据，并通过该socket对象向客户端写入数据。
        // 而只启动一个主线程去操作客户端socket对象的IO操作，这在性能上来讲是很差劲的。
        // 服务端对客户端请求的处理（读取客户端请求的数据）和堆客户端进行相应输出（向客户端写入数据）都是基于服务端在这里获取到的客户端socket对象操作的。
        // 因此，服务端不建议只使用一个线程对象去操作获取到的这个socket对象。
        // 我们可以通过在服务端委托新的线程去操作这个socket对象，将socket对象的IO操作交给子线程去管理，主线程只负责获取客户端请求的socket即可。
        // 但这样一来，当并发量很大时，服务端对于客户端请求创建的每一个连接都启动一个对象进行IO读写，会造成服务端资源浪费和内存飙升。
        client = server.accept();
        System.out.println("服务端开始执行业务...");
        try {
            // 模拟服务端重量级操作
            Thread.sleep(30000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 服务端获取客户端的输出流，向客户端返回一个 Hello World!
        String str = "Hello World!";
        out = new PrintStream(client.getOutputStream());
        out.println(str);
        out.close();
        client.close();
        server.close();
        System.out.println("服务端执行业务结束...");
    }
}

```
上面代码是一个ServerSocket在指定端口9999上监听启动的一个TCP服务端应用。   
（1）该服务端程序没有使用多线程，因此它只有一个服务端线程去执行。   
（2）该服务端模拟了一个重量级操作。   
（3）ServerSocket对象的accept()方法，当客户端在该端口发送请求进来后，可以通过该方法来创建服务端的socket对象。   
该方法接受客户端的TCP请求，创建一个服务端的Socket对象，这个Socket对象实际上就是由客户端的TCP连接转换而来的，可以将这个服务端Socket对象看做是客户端的一个TCP请求。   
通过服务端Socket对象可以获取客户端TCP请求中的信息。   
传统的accept方法造成对服务端socket对象创建的阻塞，注意accept()方法就是获取基于客户端的请求来创建对应的服务端Socket对象的。   
这是致命的，因为服务端对客户端请求的处理（读取）和对客户端进行响应输出（写入）都是基于服务端的socket对象的。   
当然可以通过多线程委托新的线程去执行读写操作。但当并发量很大时，服务端对每个连接都创建一个线程进行IO读写，会造成资源浪费和内存飙升。   
（4）服务端正常执行完毕，会获取客户端的输出流，向客户端输出一个“Hello World!”回去。   

启动服务端：   
![](/images/myBlog/2021-03-01-serversocket-run.png)   
发现服务端程序一直处于启动状态，监听客户端发送的TCP请求连接。   

## 6.2 TCP客户端
### 6.2.1 同步请求
zeh.myjavase.code28socket.demo01.TCPClientSocket
```java
package zeh.myjavase.code28socket.demo01;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.Socket;
import java.net.UnknownHostException;

import org.junit.Test;

// Socket类：TCP客户端类
public class TCPClientSocket {

    // 同步请求：客户端需要等待服务端的响应结果。
    @Test
    public void testSyn() throws UnknownHostException, IOException {
        // 声明客户端Socket对象
        Socket client = null;
        // 连接目标主机和端口
        client = new Socket("localhost", 9999);

        // 客户端超过20s后读写超时。
        client.setSoTimeout(20000);
        System.out.println("客户端开始发起请求...");

        // 声明BufferedReader对象，接收字符流信息
        BufferedReader buf = null;

        // 取得服务端返回的socket的输入流，getInputStream()方法也是个阻塞方法，它会一直阻塞客户端的运行，直到服务端响应数据回来或者客户端抛出异常。
        buf = new BufferedReader(new InputStreamReader(client.getInputStream()));
        // 从输入流中读取服务端返回的内容
        String str = buf.readLine();
        // 输出
        System.out.println("服务器端返回的内容：" + str);
        System.out.println("客户端的后续操作....");
        //先关闭流
        buf.close();
        //再关闭socket
        client.close();

        System.out.println("客户端请求结束...");
    }

}

```
客户端的getInputStream()方法用于获取服务端的数据，这个方法和服务端的accept()方法类似，也是个阻塞方法。   
它会一直阻塞客户端线程的运行，直到服务端正常返回数据或者客户端发生异常。    

启动客户端：   
![](/images/myBlog/2021-03-01-serversocket-client-run-syn.png)   
发现客户端线程已经开始运行了，打印了对应的语句。   

这时候再看下服务端的控制台：   
![](/images/myBlog/2021-03-01-serversocket-server-run-syn.png)   
发现服务端此时接收到了客户端的TCP请求，服务端线程继续向下执行，打印了“服务端开始执行业务...”这句话。   
从这可以看出，accept()方法会对服务端Socket对象的创建造成阻塞，这意味着只有客户端的TCP请求连接进来后，才会放行，否则将一直等待客户端发送TCP请求。   

我们等了20s，发现客户端竟然抛出了timeout异常：   
![](/images/myBlog/2021-03-01-serversocket-client-run-syn-timeout.png)   
这是咋回事呢？   
看我们的客户端，有这么一句话：   
```java
client.setSoTimeout(20000);
```
这表示设置客户端的读写超时时间，客户端超过20s后要没有读取到服务端返回的数据，就直接抛出timeout异常。   

此时我们看看服务端的控制台，发现没有任何变化：   
![](/images/myBlog/2021-03-01-serversocket-server-run-syn.png)   
但当再过了10s之后，起了变化：   
![](/images/myBlog/2021-03-01-serversocket-server-run-syn-finish.png)    
发现服务端从接收到客户端的请求打印了一句话“服务端开始执行业务...”，然后总共过了30s后，输出了最后一句话“服务端执行业务结束...”，这个时候服务端程序正常执行完毕，结束了监听，程序关闭了。   

我们来梳理一下，先看服务端的流程：     
（1）服务端启动运行，在9999端口上监听客户端的TCP请求，如果没有请求，就一直监听。在accept()方法执行前，输出“服务端运行，等待客户端连接...”      
（2）服务端启动后，当有客户端请求进行后，开始通过accept()方法来创建服务端的socket对象，这个方法是阻塞的，一直会等待客户端的TCP请求进来，如果同时有多个请求进来，它会阻塞执行。   
（3）当服务端监听到客户端的TCP请求进来后，accept()方法开始正常创建服务端socket，然后继续向下执行，先输出一句话 “服务端开始执行业务...”   
（4）接着服务端模拟了一个等待30秒的重量级操作，这意味着，服务端程序的业务需要执行30秒。   
（5）当30秒之后，服务端才开始获取客户端的输出流，向客户端返回一个“Hello World!”的数据。   

再看客户端的流程：   
（1）客户端启动运行，连接服务端的9999端口。   
（2）客户端设置了读取超时时间为20秒，这意味着在客户端连接到服务端之后，20秒之内如果读取不到服务端的响应数据的话，那么客户端将会抛出timeout异常。   
（3）客户端如果不超时，就能够正常读取服务端返回的内容。   
（4）但显然，客户端在连接到服务端的20秒之后，抛出了读取超时异常。因为服务端的操作需要30秒。   
（5）客户端的超时异常不会影响服务端程序的执行，当服务端程序执行了30秒之后，服务端的业务正常完毕。   

所以，上面的案例就是客户端的同步请求，意味着客户端需要等待服务端的响应结果，在这个等待的过程中，客户端线程啥都干不了。   

### 6.2.2 默认异步请求
```java
    // 默认异步请求：
    // 客户端和服务端不存在任何的数据交互（包括客户端上送数据或者接受服务端的响应数据），则请求默认就是异步请求的，实际上仅仅建立连接而不存在任何交互的请求是没有任何阻塞且没有任何意义的。
    @Test
    public void testAsyn() throws UnknownHostException, IOException {
        Socket client = null;
        client = new Socket("localhost", 9999);

        // 客户端超过20s后读写超时。
        client.setSoTimeout(20000);
        System.out.println("客户端开始发起请求...");

        client.close();

        System.out.println("客户端请求结束...");
    }
```
从上一个案例我们分析了，对于客户端来讲，如果需要获取服务端响应的数据，那么客户端发起的请求就是同步请求。   
客户端需要一直等待服务端返回的响应数据后，线程才能继续向下执行。   
这根本原因是因为通过客户端创建的socket对象连接到服务端之后，其getInputStream()方法是阻塞方法，它在服务端响应数据回来前会一直阻塞客户端线程。   

那如果客户端不需要服务端的响应数据，只是和服务端建立完连接，就关闭客户端，这时候，客户端的请求就是异步请求了。   

运行如下：   
```youtrack
客户端开始发起请求...
客户端请求结束...
```   
发现客户端直接运行结束了。   

而此时服务端在干嘛呢？   
服务端从收到客户端连接后，然后就一直在执行它的逻辑，直到30秒之后，执行完毕：   
```youtrack
服务端运行，等待客户端连接...
服务端开始执行业务...
服务端执行业务结束...
```

# 7. 服务端无限制接收客户端TCP请求
上面的案例有个缺点：那就是服务端接收一个客户端的TCP请求后，服务端就关闭了。   
在真实的场景中，服务端应该一直处于监听状态，无限制的接收客户端的TCP请求，只要客户端发起请求连接，服务端就能够接收该连接然后创建对应的服务端Socket对象。   
下面我们将编写一个Echo的案例：   
Echo程序是网络通信编程中的一个经典案例，就是客户端发给服务端什么内容，服务端会在该内容前面加上“Echo:”然后将内容返回客户端，又称之为返显或者回显。   

## 7.1 服务端无限制接收客户端请求
zeh.myjavase.code28socket.demo02.EchoServerSocket
```java
package zeh.myjavase.code28socket.demo02;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.ServerSocket;
import java.net.Socket;

// Echo程序服务端
public class EchoServerSocket {
    public static void main(String[] args) {
        // 声明ServerSocket对象
        ServerSocket server = null;
        // 声明服务端的Socket对象
        Socket client = null;
        // 声明字节打印流
        PrintStream out = null;
        // 声明字符输入流，用于接收客户端发送的消息
        BufferedReader buffer = null;
        try {
            server = new ServerSocket(9999);
            boolean flag = true;

            // 服务端无限制接收客户端的连接，而不是客户端连接一次后，服务断就关闭socket了
            while (flag) {
                System.out.println("服务端运行，等待客户端连接...");

                // 接收到客户端的连接后，获取该请求中的客户端socket
                client = server.accept();
                buffer = new BufferedReader(new InputStreamReader(
                        // 取得客户端的输入流
                        client.getInputStream()));

                // 实例化输出流，向客户端输出
                out = new PrintStream(client.getOutputStream());
                boolean f = true;

                // 客户端循环操作
                while (f) {
                    // 无限制循环读取客户端发送来的信息
                    String str = buffer.readLine();
                    System.out.println("读取到的客户端信息：" + str);
                    if (str == null || "".equals(str)) {

                        // 如果为空，终止循环，不再读取客户端内容
                        f = false;
                    } else {

                        // 如果客户端输入信息为byebye，服务器端也不再读取客户端内容
                        if ("byebye".equals(str)) {
                            f = false;
                        } else {
                            // 如果都不是，那么服务端此时向客户端回显信息
                            out.println("Echo:" + str);
                        }
                    }
                }
                out.close();
                buffer.close();

            }
            server.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

运行：   
```youtrack
服务端运行，等待客户端连接...
```
服务端将永远运行，一直等待客户端的请求连接。   

## 7.2 客户端
zeh.myjavase.code28socket.demo02.EchoClientSocket
```java
package zeh.myjavase.code28socket.demo02;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.Socket;
import java.net.UnknownHostException;

// Echo程序客户端
public class EchoClientSocket {
    public static void main(String[] args) {
        // 声明Socket对象
        Socket client = null;
        // 声明字符输入流，从键盘接收输入信息
        BufferedReader input = null;
        // 声明打印流，向服务端发送信息
        PrintStream out = null;
        // 声明字符输入流，接收服务端返回的响应信息
        BufferedReader buffer = null;

        try {
            // 指明目标服务器ip和端口并连接
            client = new Socket("localhost", 9999);
            // 从键盘接收数据到输入流
            input = new BufferedReader(new InputStreamReader(System.in));
            // 打印流
            out = new PrintStream(client.getOutputStream());
            buffer = new BufferedReader(new InputStreamReader(
                    // 输入流，获取服务端响应的数据
                    client.getInputStream()));
            boolean flag = true;

            // 客户端无限制输入信息
            while (flag) {
                System.out.println("输入信息：");
                // 从键盘读取信息
                String str = input.readLine();
                // 通过打印流将信息发送给服务端
                out.println(str);
                if ("byebye".equals(str)) {
                    // 如果键盘输入的是byebye，则客户端循环退出
                    flag = false;
                } else {
                    // 若不是byebye，则接受服务端响应的Echo信息
                    String echo = buffer.readLine();
                    System.out.println(echo);
                }
            }
            client.close();
            buffer.close();

        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
（1）客户端从键盘循环读取信息；   
（2）读取到的信息，发送给服务端程序；   
（3）如果键盘输入的是byebye，客户端程序退出；   
（4）如果是其他信息，则从服务端读取服务端返回的数据。   

运行：   
![](/images/myBlog/2021-03-01-serversocket-echo-client.png)   

此时看看服务端：   
![](/images/myBlog/2021-03-01-serversocket-echo-server.png)   
服务端刚刚处理了一个客户端的请求，它依旧保持运行状态，继续等待接收其他的TCP请求。   

此时如果我们尝试重新运行刚才的客户端：     
![](/images/myBlog/2021-03-01-serversocket-echo-client2.png)   
第二次运行客户端，输入了一堆东西。   
再看看此时的服务端：   
![](/images/myBlog/2021-03-01-serversocket-echo-server2.png)     
它在之前的基础上又打印了客户端的第二次请求，并且依旧保持运行状态，等待接收其他的客户端请求。   

## 7.3 总结
尽管上面的服务端程序使用死循环，无限制的接收任意的客户端请求，看起来就像是我们启动的tomcat容器一样，很完美。   
但是该程序有什么缺陷呢？   
尽管该服务在9999上监听并且无限制接收客户端的连接，但是该服务端缺点就是同一个时间点只能接收一个客户端的请求，而另外一个客户端要想请求的话必须得等待当前客户端请求处理完毕才可以。   
这意味着，如果同时存在多个客户端请求都连接该服务器，那么多个请求将会出现阻塞等待的现象，导致性能低下。      
解决办法：可以在服务端程序中应用多线程。   
   
# 8. 多线程服务端
## 8.1 在服务端使用多线程
先搞一个线程任务类，去包装服务端的socket（服务端创建的socket实际上是从客户端请求连接中获取到的客户端socket）   
zeh.myjavase.code28socket.demo03.EchoThread
```java
package zeh.myjavase.code28socket.demo03;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.Socket;

public class EchoThread implements Runnable {

    private Socket client = null;

    // 通过构造方法设置服务端的socket
    public EchoThread(Socket client) {
        this.client = client;
    }

    @Override
    public void run() {
        PrintStream out = null;// 定义打印流
        BufferedReader buffer = null;// 定义输入流，接收客户端发送的消息
        try {

//			try {
//				System.out.println("IO委托线程缓慢处理...");
//				Thread.sleep(20000000);
//			} catch (InterruptedException e) {
//				e.printStackTrace();
//			}

            // 打印流
            out = new PrintStream(client.getOutputStream());
            buffer = new BufferedReader(new InputStreamReader(
                    // 输入流
                    client.getInputStream()));
            boolean flag = true;
            while (flag) {
                String str = buffer.readLine();
                if (str == null || "".equals(str)) {
                    // 结束对客户端信息的读取
                    flag = false;
                } else {
                    // 如果客户端输入信息为byebye，服务器端也不再读取客户端内容
                    if ("byebye".equals(str)) {
                        flag = false;
                    } else {
                        // 如果都不是，那么服务端此时向客户端回显信息
                        out.println("Echo:" + str);
                    }
                }
            }
            out.close();
            client.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```
服务端程序：   
zeh.myjavase.code28socket.demo03.EchoServerSocketThread
```java
package zeh.myjavase.code28socket.demo03;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

// 多线程的Echo程序服务端
public class EchoServerSocketThread {
    public static void main(String[] args) {
        ServerSocket server = null;
        Socket client = null;
        try {
            // 此服务器在9999端口上监听，ServerSocket对象在客户端连接建立前一直阻塞在accept方法上
            server = new ServerSocket(9999);
            boolean flag = true;

            // 服务端无限制接收客户端的连接，而不是客户端连接一次后，服务断就关闭socket了
            while (flag) {
                System.out.println("服务端运行，等待客户端连接...");

                // accept()方法在接收到客户端请求前会一直阻塞直到接收到客户端的请求后才放行主线程。
                client = server.accept();

                // 线程对象是线程子类的载体，对于每个客户端服务端都新建一个线程对象专门负责。
                // 通过多线程，每次请求委托一个新的线程去处理IO读写。则accept()立马返回主线程去等待创建下一个连接。
                // 比如1000个客户端同时请求 ，相当于同时创建1000个线程去处理IO。
                // 但问题是，假如100万个并发量，则瞬间导致线程数飙升，内存飙升，服务器崩溃。解决办法是NIO。
                // 扩展点：主线程在accept()方法上等待客户端连接，一旦有连接进来就立即获取客户端socket对象。
                // 获取到当前客户端请求的socket对象后，委托子线程去操作该socket对象，主线程立即返回继续在端口上等待接收下一个客户端请求。
                // 这种方式在服务端而言，其实就是多线程的服务端连接，说白了就是异步连接。
                new Thread(new EchoThread(client)).start();
            }
            server.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```
上面的服务端程序，使用了多线程。   
使用多线程，并发处理多个客户端的并发访问。所谓服务端使用多线程去处理客户端请求，其实就是服务端启动多个线程，分别去获取每个客户端请求的socket对象。   
accept()方法在获取客户端socket对象的时候并不是重量级的，但通过该对象去操作IO读写有可能就是重量级的。   
因此，对于每一个客户端请求，服务端都委托一个子线程去处理这个客户端socket的IO操作，这样主线程就被腾出来了，继续在端口上监听，等待新的客户端连接。   

## 8.2 客户端程序
客户端程序还是使用Echo的客户端。   

## 8.3 运行
我们运行服务端，然后再运行客户端。   
可以看到客户端如下：   
![](/images/myBlog/2021-03-01-serversocket-server-run-asyn.png)   
此时我们再看服务端：   
![](/images/myBlog/2021-03-01-serversocket-server-run-asyn2.png)   
可以看出，客户端发送请求后，服务端里的主线程立马就接收到请求了，然后委托子线程去执行该请求，紧接着会返回等待接收新的客户端请求。   

# 9. UDP案例   
UDP server端   
zeh.myjavase.code28socket.demo04.UDPServer   
```java
package zeh.myjavase.code28socket.demo04;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

// UDP服务端
public class UDPServer {
    public static void main(String[] args) {
        DatagramSocket ds = null;// 通信套接字载体
        DatagramPacket dp = null;// 数据报载体
        try {
            ds = new DatagramSocket(3000);// 服务方在3000端口监听
            String str = "hello,Eric!";
            
            // 实例化DatagramPacket对象，指定数据内容，长度，要发送的目标地址，目标端口
            // 此时向客户端所在的9000端口发送数据
            dp = new DatagramPacket(str.getBytes(), str.length(),
                    InetAddress.getByName("localhost"), 9000);
            System.out.println("发送信息.");
            ds.send(dp);// 发送数据报
            ds.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}

```
UDP与TCP：   
TCP建立的连接是可靠的连接，客户端发送的消息服务端收到并响应后连接才断开；优点是可靠，缺点是浪费系统资源；   
UDP建立的连接是不可靠的连接，客户端只管发送消息，发完了连接就断开，不管服务方是否能够收到，UDP中所有的信息采用数据报的形式发送出去。   
DatagramSocket用来通信，既做客户端又做服务端，且客户端和服务端各自启动监听端口；   
DatagramPacket是客户端端和服务端传递的数据报。   

UDP client端   
zeh.myjavase.code28socket.demo04.UDPClient   
```java
package zeh.myjavase.code28socket.demo04;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;

// UDP客户端
public class UDPClient {
    public static void main(String[] args) {
        // 声明DatagramSocket对象
        DatagramSocket ds = null;

        // 声明接收数据的字节数组
        byte[] buf = new byte[1024];
        // 声明DatagramPacket对象
        DatagramPacket dp = null;
        try {

            // UDP客户端在9000端口监听
            ds = new DatagramSocket(9000);

            // 指定接收数据的长度为1024字节
            dp = new DatagramPacket(buf, 1024);
            System.out.println("等待接收数据....");

            // 接收数据
            ds.receive(dp);

            // 接收数据
            String str = new String(dp.getData(), 0, dp.getLength()) + "from"
                    + dp.getAddress().getHostAddress() + ":" + dp.getPort();
            System.out.println("服务端响应为：" + str);
            ds.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```
DatagramSocket用来通信，既做客户端又做服务端，且客户端和服务端各自启动监听端口；   
DatagramPacket是客户端端和服务端传递的数据报。   

运行如下：   
![](/images/myBlog/2021-03-01-udp-client.png)   
![](/images/myBlog/2021-03-01-udp-server.png)   

# 10. 得到本地和远程ip地址
zeh.myjavase.code28socket.demo05.InetAddressGetLocalAndRemoteIP
```java
package zeh.myjavase.code28socket.demo05;

import java.io.IOException;
import java.net.InetAddress;

// Java网络编程：得到本机和远程ip地址
public class InetAddressGetLocalAndRemoteIP {
    public static void main(String args[]) throws IOException {

        // 主要依靠主机类InetAddress得到主机对象
        InetAddress locAdd = null;
        // InetAddress remAdd = null;
        locAdd = InetAddress.getLocalHost();
        // remAdd = InetAddress.getByName("www.baidu.com");
        String locHostName = locAdd.getHostName();

        // 主机对象得到主机名称
        // String remHostName = remAdd.getHostName();
        String locHostAddress = locAdd.getHostAddress();

        // 主机对象得到主机ip地址
        // String remHostAddress = remAdd.getHostAddress();
        System.out.println("本机主机名称：" + locHostName + ";本机IP地址："
                + locHostAddress);
        // System.out.println("远程主机名称：" + remHostName + ";远程IP地址：" +
        // remHostAddress);
        System.out.println("本机是否可以到达？" + locAdd.isReachable(5000));
    }
}

```

# 11. URLConnection类
```java
package zeh.myjavase.code28socket.demo06;

import java.io.IOException;
import java.net.URL;
import java.net.URLConnection;

// Java网络编程：URLConnection类取得网络资源的常见属性
public class URLConnectionGetRemoteSourceAttribute {
    public static void main(String args[]) throws IOException {
        URL url = new URL(
                "http://138.138.2.241:8081/esbconsole/esb/public/login.jsp");
        URLConnection urlCon = url.openConnection();// 通过url对象获得统一资源定位符
        System.out.println("网络资源的内容大小：" + urlCon.getContentLength());
        System.out.println("网络资源的内容类型：" + urlCon.getContentType());
        System.out.println("网络资源的编码类型：" + urlCon.getContentEncoding());
    }
}

```
URLConnection类：统一资源定位符连接类，可以取得网络资源的属性信息   

# 12. 编码类和解码类
zeh.myjavase.code28socket.demo07.URLEncoderAndURLDecoder
```java
package zeh.myjavase.code28socket.demo07;

import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.net.URLEncoder;

// Java网络编程：编码类和解码类
public class URLEncoderAndURLDecoder {
    public static void main(String args[]) throws UnsupportedEncodingException {
        
        String keyword = "Eric 赵二虎";
        String encodeByU8 = URLEncoder.encode(keyword, "UTF-8");
        String decodeByU8 = URLDecoder.decode(encodeByU8, "UTF-8");
        System.out.println("编码后：" + encodeByU8);
        System.out.println("解码后：" + decodeByU8);

        String encodeByISO = URLEncoder.encode(keyword, "ISO-8859-1");
        String decodeByISO = URLDecoder.decode(encodeByISO, "ISO-8859-1");
        System.out.println("编码后：" + encodeByISO);
        System.out.println("解码后：" + decodeByISO);

        String format = new String(keyword.getBytes("ISO-8859-1"), "utf-8");
        System.out.println("format:" + format);
    }
}

```   
编码类：URLEncoder类   
解码类：URLDecoder类   
编码问题存在两个方面：JVM之内和JVM之外。   
1、Java文件编译后形成class   
这里Java文件的编码可能有多种多样，但Java编译器会自动将这些编码按照Java文件的编码格式正确读取后产生class文件，这里的class文件编码是Unicode编码（具体说是UTF-16编码）。   
因此，在Java代码中定义一个字符串：    
```java
String s="汉字";
```
不管在编译前java文件使用何种编码，在编译后成class后，他们都是一样的----Unicode编码表示。   
2、JVM中的编码   
JVM加载class文件读取时候使用Unicode编码方式正确读取class文件，那么原来定义的String s="汉字";在内存中的表现形式是Unicode编码。      

# 13. URL类取得网络资源
zeh.myjavase.code28socket.demo08.URLGetRemoteSource
```java
package zeh.myjavase.code28socket.demo08;

import java.io.IOException;
import java.io.InputStream;
import java.net.URL;
import java.util.Scanner;

// Java网络编程：URL类取得网络资源
public class URLGetRemoteSource {
    public static void main(String args[]) throws IOException {
        URL url = new URL("http://138.138.2.241:8081/esbconsole/esb/public/login.jsp");

        // 取得网络资源输入流，把网络资源读取到输入流中
        InputStream in = url.openStream();

        // 从输入流中读取数据并验证
        Scanner scan = new Scanner(in);
        scan.useDelimiter("\n");
        while (scan.hasNext()) {
            System.out.println(scan.next());
        }
        scan.close();
    }
}

```
URL类：统一资源定位符类，可以取得网络资源   


