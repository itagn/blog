作者：蔡东-uestc-2017届
转载请注明出处并保留原文链接

## 简介项目

    这是我的毕业设计，基于DES加密的即时通信聊天系统。
    Java swing写ui界面，java写服务器，mysql做数据库，tcp的socket通信。
    实现功能：
    1.登录注册
    2.用户添加好友、删除好友
    3.用户在线、离线发送消息
    4.用户在线、离线发送文件
    5.用户添加好友和用户发送文件的权限请求
    6.未读消息的声音和颜色提示
    7.des加密解密保证信息在网络传输中的安全
    8.用户上线离线过程，实现客户端和服务器的实时刷新
    9.通过继承Thread实现多线程编程

![exe](https://github.com/itagn/blog/raw/master/InstantMessageSystem/img/exe.png)
## socket套接字原理以及传输
    socket是一对套接字，客户端和服务器的一对配对socket，即socket和serverSocket。建立serverSocket(Int serverPort)监听服务器的端口号，然后创建客户端的套接字进行绑定服务器socket(String serverIP, Int serverPort)。
一、通过客户端向服务器发送字符串

![chat](https://github.com/itagn/blog/raw/master/InstantMessageSystem/img/chat.png)
客户端发送消息，可以接受服务器返回的消息
```java
//  客户端代码
Socket s = new Socket(serverIp, serverPort);
DataInputStream input = new DataInputStream(new BufferedInputStream(s.getInputStream()));
DataOutputStream output = new DataOutputStream(new BufferedOutputStream(s.getOutputStream()));
output.writeUTF('hello server!');
input.readUTF();  // hello client!
output.close();
s.close();
```
服务器接收到消息，可以返回信息
```java
//  服务器代码
ServerSocket ss = new ServerSocket(serverPort);
//  一直制监听客户端不同时间传来的数据
while(true){
    Socket s = ss.accept();
    DataInputStream input = new DataInputStream(new BufferedInputStream(s.getInputStream()));
    DataOutputStream output = new DataOutputStream(new BufferedOutputStream(s.getOutputStream()));
    input.readUTF();  // hello server!
    output.writeUTF('hello client!);
}
```

二、通过客户端向服务器发送文件
    
    发送文件的思维就是tcp的三次握手，步骤如下：
    1.告诉服务器，客户端要发送文件
    2.服务器开始监听数据流，并返回信息告诉客户端准备好了
    3.客户端开始传输数据流，客户端开始接收数据流，接收完数据流后断开socket通信，传输完毕

![file](https://github.com/itagn/blog/raw/master/InstantMessageSystem/img/file.png)
客户端发送文件数据流
```java
//  sendPath为要发送的文件路径
DataInputStream fis = new DataInputStream(new BufferedInputStream(new FileInputStream(sendPath)));
DataOutputStream ps = new DataOutputStream(s.getOutputStream());
int sendSize = 8192;  //  设置8192大小缓存
byte[] sendBuf = new byte[sendSize];
 while (true) {
     int read = 0;
     if (fis != null) {
         read = fis.read(sendBuf);
     }
     if (read == -1) {
         break;
     }
     ps.write(sendBuf, 0, read);
 }
 ps.flush();
 ps.close();
 fis.close();
```
服务器接收文件数据流
```java
//  savePath为要保存的文件路径
DataOutputStream fileOut = new DataOutputStream(new BufferedOutputStream(new BufferedOutputStream(new FileOutputStream(savePath))));
DataInputStream clin = new DataInputStream(new BufferedInputStream(s.getInputStream()));
int getSize = 8192;
byte[] getbuf = new byte[getSize];
while (true) {
    int read = 0;
    if (clin != null) {
        read = clin.read(getbuf);
    }
    if (read == -1) {
        break;
    }
    fileOut.write(getbuf, 0, read);
}
fileOut.close();
```
三、多个客户端之间通信
    
    原理很简单，服务器做到的是转发的作用，服务器接收到client1的请求后，检查client2是否在线。如果client2在线，从数据库获取client2的ip地址和端口号，此时服务器做客户端，client2做服务器，向client2发送请求。
    具体逻辑如下：
    client1 -> server1;
    server1 -> client && client2 -> server;
    client -> server; 

## 在线和离线发送的区别
一、在线和离线发送消息

    发送消息前经过DES加密后发送给服务器，服务器判断接收方是否在线。
    如果接收方在线，则转发信息，接收方接收到消息后，DES解密后显示在界面。
    如果接收方离线，储存在数据库，并设置未读信息标记，接收方上线后会检查自己是否存在离线消息（未读消息），如果存在离线消息则发送数据给接收方，界面语言和颜色提示。
    
二、在线和离线接收消息

    发送文件，通过之前的顺序直接发送给服务器，服务器判断接收方是否在线。
    如果接收方在线，则发送一个接收文件的权限请求给接收方，如果接收方拒绝接收文件，服务器删除文件。如果接收方同意接收文件，则转发文件，服务器发送成功后删除文件。
    如果接收方离线，设置未接受文件标记，接收方上线后会检查自己是否存在未接受文件，如果存在，执行上面的操作。

## 添加好友和删除好友
![friends](https://github.com/itagn/blog/raw/master/InstantMessageSystem/img/friends.png)
一、添加好友

    搜索好友的网名后，点击添加，不存在或者错误，会弹窗提示。存在且不是好友，会发送添加好友的权限请求给接收方，接收方同意添加好友后，从服务器获取资源，刷新双方的好友界面，否则不进行操作。
    
二、删除好友
    
    输入好友的网名，点击删除，不存在或者错误，会弹窗提示。存在并是好友，则直接删除好友。

## 遇到并解决的问题
一、Java swing在多个页面聊天窗口打开的时候，接收到的消息找不到对应的聊天窗口。
    
    使用array数组来储存client1与client2的聊天窗，通过构造函数参数传值，来寻找到对应的窗口，并将信息绘画在聊天窗
    
二、Java swing在窗口最小化之后，打开后界面是空白

    通过监听界面，如果页面获取了焦点，则重绘界面，解决最小化之后，打开是空白的问题

三、如何解决用户上下线实时更新

    用户上线和离线的时候、服务器界面进行重绘界面，并通知客户端请求资源后重绘
    
四、动态的绑定服务器地址

    服务器ip地址储存在外部文件，可以修改外部文件来绑定服务器ip地址，服务器可以迁移
    
    
## 未解决的问题
一、聊天系统局限于局域网

    在获取ip地址时，获取到的ip地址是路由器的局域网地址，外网ip地址将不能找到。
    
二、保存聊天记录

    关闭聊天窗口之后打开聊天窗口，页面将被清空，而且找不到之前的聊天记录
    
三、发送文件过程中网络断开，无法继续之前已发送的部分继续发送

    可能程序会卡住，之后需要重新发送，大文件发送不友好

四、DES加密的密钥固定

    所有聊天信息传输过程中，都是会用同一套密钥，安全性降低
    
## 比较丑的界面

![login](https://github.com/itagn/blog/raw/master/InstantMessageSystem/img/login.png)
![register](https://github.com/itagn/blog/raw/master/InstantMessageSystem/img/register.png)
![server](https://github.com/itagn/blog/raw/master/InstantMessageSystem/img/server.png)



作者：微博 [@itagn][1] - Github [@itagn][2]

[1]: https://weibo.com/p/1005053782707172
[2]: https://github.com/itagn