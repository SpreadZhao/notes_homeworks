使用java自己搭建的http服务器，这部分的教学来自这个网站：

[A Simple HTTP Server in Java (commandlinefanatic.com)](https://commandlinefanatic.com/cgi-bin/showarticle.cgi?article=art076)

# 1. 小爱同学

但是在开始这个项目之前，我们先来看一个比较简单的，也是我之前做过的一个项目。这个项目模拟了智能语音助手的简单工作方式，其实就是服务端在监听，客户端去发送信息，当服务端检测到信息之后，传回对应的对象即可。

我们使用的是流来进行数据交互，核心的方法是下面两个：

```java
readObject()
writeObject()
```

因此我们的数据也是`Object`类型。现在就从服务端开始来逐个解释。

首先，服务端需要监听一个指定的端口，那么我们用socket做到这一点呢？有一个`ServerSocket`类，这个类中有一个`accept`方法，官方的描述是这样的：

![[Pasted image 20221029185127.png]]

所以这就是两个socket之间的第一次交互。首先是客户端的socket，当它想要尝试登陆(如何登陆之后再说)某一个服务端时，首先要见一见服务端的socket，而这个方法就会在服务端那边得到客户端socket的实例。而当服务端这里啥事没有的时候，就会一直阻塞着，直到有人来为止。那么我们服务端代码的写法就已经很清晰了：

```java
public class Server {  
    private int port;  
    private ServerSocket serverSocket;  
    private boolean isRunning;  
  
    public Server(int port){  
        this.port = port;  
        isRunning = true;  
    }  
  
    public void start(){  
        try{  
            serverSocket = new ServerSocket(port);  
            System.out.println("[server.Server]started, listening port: " + serverSocket.getLocalPort());  
            while (isRunning){  
                Socket clientSocket = serverSocket.accept();  
                System.out.println("[server.Server] " + clientSocket.getRemoteSocketAddress() + " connect successfully!");  
                new Thread(new ClientSocketHandlingTask(clientSocket)).start();  
            }  
        }catch (IOException e){  
            e.printStackTrace();  
        }  
    }  
}
```

这里唯一我们还不清晰的，是这句话：

```java
new Thread(new ClientSocketHandlingTask(clientSocket)).start();
```

这句话新建了一个`Runnable`的子类，并起了一个线程去运行它，而这就是服务端在成功得到一个client的socket之后做的事，那么这个`Runnable`肯定就是和客户端交互的逻辑了。我们接下来就来看看其中的细节：

```java
public class ClientSocketHandlingTask implements Runnable{  
  
    private Socket clientSocket;  
    private ObjectInputStream inputStream;  
    private ObjectOutputStream outputStream;  
    private String clientName;  
    private static int id = 0;  
  
    public ClientSocketHandlingTask(Socket socket){  
        this.clientSocket = socket;  
        this.clientName = "[user " + ++id + "]";  
        try {  
            inputStream = new ObjectInputStream(clientSocket.getInputStream());  
            outputStream = new ObjectOutputStream(clientSocket.getOutputStream());  
        } catch (IOException e) {  
            throw new RuntimeException(e);  
        }  
    }  
  
    @Override  
    public void run() {  
        Object msg = null;  
        try {  
            while((msg = inputStream.readObject()) != null){  
                System.out.println("[Server]receive " + clientName + "'s message: " + msg);  
            }  
        }  catch (ClassNotFoundException | IOException e) {  
            e.printStackTrace();  
        }  
    }  
}
```

在构造这个任务的时候，我们最重要的任务就是拿到客户端的输入流和输出流，借助这两个流我们就能从客户端读信息和向客户端写信息了。

当这个任务被执行的时候，它会调用输入流的`readObject`方法(注意，客户端的输入流是服务端的输出流)，为了弄清楚这段代码到底是怎么运行的，我重写了一下：

```java
public void run() {  
    Object msg = null;  
    try {  
        System.out.println("haha");  
        while(true){  
            msg = inputStream.readObject();  
            System.out.println("[Server]test");  
            if(msg != null){  
                System.out.println("[Server]receive " + clientName + "'s message: " + msg);  
            }else{  
                System.out.println("[Server]msg is null");  
                break;            }  
        }  
    }  catch (ClassNotFoundException | IOException e) {  
        e.printStackTrace();  
    }  
}
```

我看到的执行结果是这样的：

```shell
[Server]enter port to listen: 1234
[server.Server]started, listening port: 1234
[server.Server] /127.0.0.1:50526 connect successfully!
haha
[Server]test
[Server]receive [user 1]'s message: asdf
[Server]test
[Server]receive [user 1]'s message: fgh
```

因此我们能推测出来：**`readObject`是一个阻塞方法**，只有收到了客户端发来的消息时才会继续执行。接下来就是客户端的代码了，这部分代码非常简单：

```java
public class client {  
    public static void main(String[] args) {  
        Scanner input = new Scanner(System.in);  
        Socket socket = null;  
        ObjectOutputStream outputStream = null;  
        ObjectInputStream inputStream = null;  
        String msg;  
  
        try {  
            System.out.print("[Client]Enter port to log in: ");  
            socket = new Socket("localhost", input.nextInt());  
            System.out.println("[Client]connection success!");  
  
            outputStream = new ObjectOutputStream(socket.getOutputStream());  
            inputStream = new ObjectInputStream(socket.getInputStream());  
  
            while(true){  
                System.out.print("[Client]please enter: ");  
                msg = input.next();  
                outputStream.writeObject(msg);  
                outputStream.flush();  
            }  
        } catch (IOException e) {  
            throw new RuntimeException(e);  
        } finally {  
            try {  
                assert inputStream != null;  
                inputStream.close();  
                outputStream.close();  
                input.close();  
            } catch (IOException e){  
                e.printStackTrace();  
            }  
        }  
    }  
}
```

最关键的还是这两行代码：

```java
outputStream.writeObject(msg);  
outputStream.flush();
```

接下来，如果客户端想要接到服务端返回的数据，聪明的你肯定也已经想到了：继续调用这个阻塞的函数等待服务端返回即可！这里我们就只给我之前写的例子了：

```java
Object returnMsg = inStream.readObject();
//如果返回值是文件，播放文件
if(returnMsg instanceof File) {
	new Thread(new PlayMusicTask((File)returnMsg)).start();
}else if("bye".equals(returnMsg)){
	break;
}else {
	System.out.println("[客户端]小爱说：" + returnMsg);
}
```

# 2. 开始搭建

有了这个小爱同学的例子，我们已经对socket编程有了一个最最基本的认识。那么接下来就开始用java来手撸一个服务器罢！

首先是最基础的类：`HttpServer`，这个类做的事就和小爱同学中的`Server`是一样的——**去监听一个端口，并等待客户登入**。