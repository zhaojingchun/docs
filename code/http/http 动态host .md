## java http请求代码指定host

[TOC]

### 简介

最近做的需求，同一个接口调用到不同的环境 ，不同环境接口域名不同还需要指定host ，每次加环境都需要修改线上服务器host，重启线上机器，就想能不能通过代码来绑定host一搜索网上就要线程方法，整理备用。

### 方法一：直接修改请求头

[参考](https://blog.csdn.net/zlfprogram/article/details/79030217)

每一个环境都有跳板机，跳板机是通过域名分发的。 nginx上只配置了域名的分发，没有配置ip分发，

于是在http请求上做了点处理，url配置ip，同时配置http头部的Host参数为该域名，HttpURLConnection 有setRequestProperty(key,value)方法来设置http头部通过ip访问，设置Host头部来让nginx识别，然后分发到相应的处理程序

```java
HttpURLConnection urlConn = null;
String url = "http://192.168.1.120/login?platform=xxx&type=account";
URL destURL = new URL(url);
urlConn = (HttpURLConnection) destURL.openConnection();
urlConn.setRequestProperty("Host", "auth.xxx.com");
```

请求发出后，404说明客户端请求路径不存在，推测估计是Host没有生效。

在网上搜了搜，找了HttpURLConnection的sun的源代码

http://javasourcecode.org/html/open-source/jdk/jdk-6u23/sun/net/www/protocol/http/HttpURLConnection.java.html

发现里面 有一个 allowRestrictedHeaders 这个参数，原来api在设计的时候，可能为了安全，限制了程序能够使用的Http Header
如，下面的都是限制的

```java
private static final String[] restrictedHeaders = {
    /* Restricted by XMLHttpRequest2 */
    //"Accept-Charset",
    //"Accept-Encoding",
    "Access-Control-Request-Headers",
    "Access-Control-Request-Method",
    "Connection", /* close is allowed */
    "Content-Length",
    //"Cookie",
    //"Cookie2",
    "Content-Transfer-Encoding",
    //"Date",
    "Expect",
    "Host",
    "Keep-Alive",
    "Origin",
    // "Referer", 
    // "TE",
    "Trailer",
    "Transfer-Encoding",
    "Upgrade",
    //"User-Agent",
    "Via"
    };

    allowRestrictedHeaders = ((Boolean)java.security.AccessController.doPrivileged(
        new sun.security.action.GetBooleanAction(
            "sun.net.http.allowRestrictedHeaders"))).booleanValue();
```

里面就有我们要设置的Host，找到原因，就好解决了，只要告诉程序，允许能使用这些限制的头部即可

```java
System.setProperty("sun.net.http.allowRestrictedHeaders", "true");
```

### 方法二、通过代理访问

[参考1](https://blog.csdn.net/goldenfish1919/article/details/8789604)

[参考2](https://www.cnblogs.com/yangzhilong/p/6640207.html)

需求：因为测试服务器有多台，一般都是通过绑定host来访问某一台测试服务器，现在就想用代码来实现绑定host的功能，这样就省去了修改本机host的步骤。
以下是用代理来实现：

```java
public static HttpURLConnection getConnection(String urlstr, String proxyhost)throws Exception {
    URL url = new URL(urlstr);
    byte ip[] = new byte[4];
    String arr[] = proxyhost.split("\\.");
    for (int i = 0; i < 4; i++) {
        int tmp = Integer.valueOf(arr[i]);
        ip[i] = (byte) tmp;
    }
    //生成proxy代理对象，因为http底层是socket实现
    Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress(
            InetAddress.getByAddress(ip), 80));
    return (HttpURLConnection) url.openConnection(proxy);
}
public static void main(String[] args) throws Exception {
    String url = "http://www.xujsh.com/api/hao123";
    String proxyhost = "192.168.119.182";
    HttpURLConnection httpUrlConn = getConnection(url, proxyhost);
    InputStream in = httpUrlConn.getInputStream();
    String response = FileUtil.getContent(in, "UTF-8");
    System.out.println(response);
}
```

参考：http://blog.csdn.net/zhongweijian/article/details/7619453

以下是基于HttpClient的实现：

```java
public static void main(String[] args)throws Exception {
    HttpHost proxyhost = new HttpHost("192.168.119.182", 80, "http");
    DefaultHttpClient httpclient = new DefaultHttpClient();
    try {
        httpclient.getParams().setParameter(ConnRoutePNames.DEFAULT_PROXY, proxyhost);
 
        HttpHost targethost = new HttpHost("www.xujsh.com", 80, "http");
        HttpGet get = new HttpGet("/api/hao123");
        
        System.out.println("executing request to " + targethost + " via " + proxyhost);
        HttpResponse rsp = httpclient.execute(targethost, get);
        HttpEntity entity = rsp.getEntity();
 
        System.out.println("----------------------------------------");
        System.out.println(rsp.getStatusLine());
        Header[] headers = rsp.getAllHeaders();
        for (int i = 0; i<headers.length; i++) {
            System.out.println(headers[i]);
        }
        System.out.println("----------------------------------------");
        if (entity != null) {
            System.out.println(EntityUtils.toString(entity));
        }
    } finally {
        httpclient.getConnectionManager().shutdown();
    }
}
```



其实也不难理解，原理大概就是：客户端发送一个请求给服务端，如果服务端是一个代理服务器，它就会转发客户端的请求，如果服务端不是一个代理服务器，
而只是一台普通的应用服务器，它就是自己来响应这个请求。

其实代理服务器本身也不是什么神秘的东西，我们来自己写一个简单的代理服务器。在本机的8989端口接收客户端的请求，然后进行转发。
服务端：

```java
public class MyProxyServer {
    public static void main(String[] args) {
        try {
            new Thread(new SimpleProxyServer(8989)).start();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
class SimpleProxyServer implements Runnable {
    private ServerSocket listenSocket;
    public SimpleProxyServer(int port) throws IOException {
        this.listenSocket = new ServerSocket(port);
    }
    public void run() {
        for (;;) {
            try {
                Socket clientSocket = listenSocket.accept();
                System.out
                        .println("Create a new Thread to handle this connection");
                new Thread(new ConnectionHandler(clientSocket)).start();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
class ProxyCounter {
    static int sendLen = 0;
    static int recvLen = 0;
    public static void showStatistics() {
        System.out.println("sendLen = " + sendLen);
        System.out.println("recvLen = " + recvLen);
    }
}
 
// must close sockets after a transaction
class ConnectionHandler extends ProxyCounter implements Runnable {
    private Socket clientSocket;
    private Socket serverSocket;
    private static final int bufferlen = 1024*1024;
    public ConnectionHandler(Socket clientSocket) {
        this.clientSocket = clientSocket;
    }
    public void run() {
        // receive request from clientSocket,
        // extract hostname,
        // create a serverSocket to communicate with the host
        // count the bytes sent and received
        try {
            byte[] buffer = new byte[bufferlen];
            int count = 0;
            InputStream inFromClient = clientSocket.getInputStream();
            count = inFromClient.read(buffer);
            String request = new String(buffer, 0, count);
            System.out.println("request:\n"+request);
            String host = extractHost(request);
            System.out.println("host:\n"+host);
            // create serverSocket
            Socket serverSocket = new Socket(host, 80);
            // forward request to internet host
            OutputStream outToHost = serverSocket.getOutputStream();
            outToHost.write(buffer, 0, count);
            outToHost.flush();
            sendLen += count;
            showStatistics();
            // forward response from internet host to client
            InputStream inFromHost = serverSocket.getInputStream();
            OutputStream outToClient = clientSocket.getOutputStream();
            while (true) {
                count = inFromHost.read(buffer);
                System.out.println("count:"+count);
                if (count < 0)
                    break;
                outToClient.write(buffer, 0, count);
                outToClient.flush();
                recvLen += count;
                showStatistics();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if(clientSocket != null)clientSocket.close();
                if(serverSocket != null)serverSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    private String extractHost(String request) {
        int start = request.indexOf("Host: ") + 6;
        int end = request.indexOf('\n', start);
        String host = request.substring(start, end - 1);
        return host;
    }
}
```

参考：http://my.oschina.net/u/158589/blog/64643
客户端测试：

```java
public class MyProxyClient {
    public static void main(String[] args) throws Exception {
        Socket socket = new Socket("localhost",8989);
        OutputStream out = socket.getOutputStream();
        out.write("GET / HTTP/1.1\nHost: www.baidu.com \n\n".getBytes());
        out.flush();
        InputStream in = socket.getInputStream();
        byte[] arr = FileUtil.readAsByteArray(in);
        System.out.println(new String(arr));
        socket.close();
    }
}
```

