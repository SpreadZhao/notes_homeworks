---
title: "2.2 Interlude: Process API"
order: "3"
---
[[Study Log/os_study/0_ostep_index|Return to Index]]

## 2.2 Interlude: Process API

介绍三个OS提供的和进程相关的API：

- fork
- exec
- wait

### 2.2.1 fork

看这个程序：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    printf("hello world (pid: %d)\n", (int)getpid());
    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        printf("hello, I am child (pid: %d)\n", (int)getpid());
    } else {
        printf("hello, I am parent of %d (pid: %d)\n", rc, (int)getpid());
    }
    return 0;
}
```

运行结果：

```shell
❯ ./p1
hello world (pid: 16066)
hello, I am parent of 16067 (pid: 16066)
hello, I am child (pid: 16067)
```

有几点需要注意。首先是我们能观察出，**新诞生的进程似乎是原来进程的copy**。但是，这个新进程却没有从main开始执行，而是好像已经调用过`fork()`了一样。

因此，这个新进程并不是完全的copy。尽管这个进程已经有了和原来进程一样的地址空间（包括似有内存）和寄存器、自己的PC（Program Counter）等等，但是这个新进程的`fork()`的返回值是不一样的：

- 父进程接收到了子进程的PID，体现在变量rc；
- 子进程接收到了0。

通过fork的这个机制，我们就能处理父进程和子进程的不同行为。

另外，运行结果中的后两句的顺序也不是确定的。这主要取决于之后会介绍的Scheduler。

### 2.2.2 wait

程序稍微改一下，让父进程的那段话永远比子进程的晚：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
    printf("hello world (pid: %d)\n", (int)getpid());
    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        printf("hello, I am child (pid: %d)\n", (int)getpid());
    } else {
        int rc_wait = wait(NULL);
        printf("hello, I am parent of %d (rc_wait: %d) (pid: %d)\n", rc, rc_wait, (int)getpid());
    }
    return 0;
}
```

这里的wait就会等待子进程结束，结束了之后才会继续执行下面的代码。

> [!note]
> 这里和线程做一个对比：
> 
> - C中的主进程默认不会等待子进程结束；
> - 调用了`wait()`之后才会等待；
> - C中的线程的行为也是一样的。main线程不会等待其它线程是否结束，除非调用join；
> - 而java中不一样，只有所有非守护线程结束之后程序才会退出。
> 
> 另外可参考：[[Study Log/android_study/android_diary/2024-02-21-android-dev-trouble#^6d7bc5|2024-02-21-android-dev-trouble]]

然而，wait不是必然等待进程结束之后才返回的。实际上，它等待的是子进程状态的变化。参照[manual page](https://man7.org/linux/man-pages/man2/waitpid.2.html)：

> If a child has already changed state, then these calls return immediately.  Otherwise, they block until either a child changes state or <u>a signal handler interrupts the call</u> (assuming that system calls are not automatically restarted using the `SA_RESTART` flag of [sigaction(2)](https://man7.org/linux/man-pages/man2/sigaction.2.html)).

意思是说，如果一个Signal Handler中断了wait，那么wait也会停止。这里的Signal Handler就大概是指，比如你向调用wait的进程发送一个sigkill，那么就肯定没了。

- [ ] #TODO tasktodo1731343577514 这个等待中断的行为有什么用？举个例子。 ➕ 2024-11-12 🔽 🆔 067jos 

### 2.2.3 exec

fork的缺点很明显：新进程只能执行和原来进程一样的代码。要想执行不同的代码，就可以用exec了。下面是一个例子：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
    printf("hello world (pid:%d)\n", (int)getpid());
    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        printf("hello, I am child (pid:%d)\n", (int) getpid());    
        char *myargs[3];
        myargs[0] = strdup("wc");
        myargs[1] = strdup("../p3.c");
        myargs[2] = NULL;
        execvp(myargs[0], myargs);
        printf("this shouldn't print out");
    } else {
        int rc_wait = wait(NULL);
        printf("hello, I am parent of %d (rc_wait:%d) (pid:%d)\n", rc, rc_wait, (int)getpid());
    }
    return 0;
}
```

这里的修改是给子进程调用execvp，并传入执行的程序和参数。结果如下：

```shell
hello world (pid:23690)
hello, I am child (pid:23691)
 26  86 750 ../p3.c
hello, I am parent of 23691 (rc_wait:23691) (pid:23690)
```

注意的是，这里是fork和exec配合使用。用fork产生进程，用exec修改子进程执行的指令。这里exec做的事情是，加载我们传入的程序和参数，覆盖掉原来fork复制出来的代码段和静态数据；并且把堆空间和栈空间和这个程序的其它空间都给重新初始化。

因此可以发现，**exec不会创建一个新的进程**，而是把当前运行的进程（fork出来的子进程）变成一个完全不同的进程。如果exec调用成功了，那么它是永远也不会返回的。

### 2.2.4 Why? Motivating the API

上面最大的问题估计就是：为啥fork，exec的设计这么奇怪？一个会复制进程，一个会修改进程还不会返回。*为什么这两个东西不能绑在一起，比如像`pthread_create()`一样，创建的同时直接把它该做什么也传进去不好吗*？

这里要拆开，最大的原因是为了构建UNIX shell。我们把fork和exec分开，就能**在fork和exec之间做一些可以定制的东西**。比如我们可以修改将要运行的程序的环境变量等等。

当我们在shell中输入了命令（比如`vim newfile.txt`）的时候：

1. shell会找到这个可执行的文件在文件系统的哪里；
2. 调用fork去创建一个新的进程；
3. 调用exec（家族中的某个函数）去修改进程，运行我们输入的命令；
4. 调用wait等待程序结束；
5. 程序结束时，shell会从wait返回并再次打印出一个prompt，等待你再次输入。

那么我们在fork和exec之间能做的东西有什么呢？举个例子，我们输入：

```shell
wc p3.c > newfile.txt
```

很简单，把`p3.c`中的内容redirect到`newfile.txt`里面。那么这里shell实际上是怎么做的呢？在上面的2 3步之间，也就是**创建出子进程，但是还没调用exec的时候**，这样做：**关闭标准输出流，并打开文件`newfile.txt`**。这样，本来wc的输出要到标准输出的。但是因为我们关闭了，就跑到文件里去了。

下面这个程序做的是同样的事情：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <sys/wait.h>
#include <sys/stat.h>

int main(int argc, char *argv[]) {
    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        close(STDOUT_FILENO);
        open("./p4.output", O_CREAT|O_WRONLY|O_TRUNC, S_IRWXU);
        char *myargs[3];
        myargs[0] = strdup("wc");
        myargs[1] = strdup("../p4.c");
        myargs[2] = NULL;
        execvp(myargs[0], myargs);
    } else {
        int rc_wait = wait(NULL);
    }
    return 0;
}
```

最关键的就是这两句：

```c
close(STDOUT_FILENO);
open("./p4.output", O_CREAT|O_WRONLY|O_TRUNC, S_IRWXU);
```

之所以能实现这样的功能，是因为OS管理文件描述符（file descriptor）的方式：**UNIX系统会从0开始找空闲的描述符**。因为我们关闭了stdout，所以此时stdout就是空闲的，所以它接下来就被新打开的文件指向了（通过调用open）。这样之后本来输出到终端的内容（比如调用`printf()`或者就是程序`wc`）就会跑到新打开的文件中了。 ^9219fd

- [ ] #TODO tasktodo1731343932699 这里“空闲”的描述我总感觉有歧义。有时间check一下。 ➕ 2024-11-12 🔽 🆔 ktkqpj 

> [!note]
> `STDOUT_FILENO`和`stdout`的区别：[c - What's the difference between stdout and STDOUT_FILENO? - Stack Overflow](https://stackoverflow.com/questions/12902627/whats-the-difference-between-stdout-and-stdout-fileno)。stdout是一个`FILE*`一个IO流；而`STDOUT_FILENO`是一个int，就是1，标准输出的编号。编号更底层，使用只能用系统调用（比如close）。而`stdout`就可以用一些库函数（比如stdio），因此，上面的程序中，close那里可以换成：
> 
> ~~~c
> fclose(stdout);
> ~~~

### 2.2.5 Process Control and Users

除了fork, exec, wait，还有很多接口可以和UNIX进程交互。比如kill，可以用来给进程发信号。比如在shell里按ctrl-c就可以发送SIGINT，ctrl-z可以发送SIGTSTP（之后可以用fg命令恢复）。

使用signal系统调用可以处理这些信号。

那么显然这里有个问题：谁能发这些信号？这要提到用户的概念。当用户登录系统之后，他自己创建出来的进程就可以被他完全控制（暂停，杀死等等）。而把系统资源（CPU，内存，硬盘等等）分发给每个用户来维持运行就是OS的责任了。

### 2.2.6 Homework

#### 2.2.6.1 Simulation

[ostep-homework/cpu-api at master · remzi-arpacidusseau/ostep-homework](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-api)

使用方式很简单：

```shell
❯ ./fork.py -s 4

ARG seed 4
ARG fork_percentage 0.7
ARG actions 5
ARG action_list 
ARG show_tree False
ARG just_final False
ARG leaf_only False
ARG local_reparent False
ARG print_style fancy
ARG solve False

                           Process Tree:
                               a

Action: a forks b
Process Tree?
Action: a forks c
Process Tree?
Action: b forks d
Process Tree?
Action: d EXITS
Process Tree?
Action: a forks e
Process Tree?
```

一开始输出的不用管，从Process Tree开始看。一开始只有一个进程是a，相当于UNIX中的init进程。之后，会不断进行每一个Action，然后它希望你画出每个Action之后的tree是什么样子的。

这里很简单，就直接给答案了，使用`-c`：

```shell
❯ ./fork.py -s 4 -c
                           Process Tree:
                               a

Action: a forks b
                               a
                               └── b
Action: a forks c
                               a
                               ├── b
                               └── c
Action: b forks d
                               a
                               ├── b
                               │   └── d
                               └── c
Action: d EXITS
                               a
                               ├── b
                               └── c
Action: a forks e
                               a
                               ├── b
                               ├── c
                               └── e
```

还有一个问题是如果我们杀掉了有儿子的进程会怎么样。这个是通过`-R`来控制的。如果不加`-R`的话，那些孤儿会被托管给初始进程a；如果加上的话，会托管给被杀进程的父亲。 ^f8d31e

下面是问题了。

> [!question]- 1\. Run `./fork.py -s 10` and see which actions are taken. Can you predict what the process tree looks like at each step? Use the -c flag to check your answers. Try some different random seeds (-s) or add more actions (-a) to get the hang of it.
> 
> ~~~shell
> ❯ ./fork.py -s 10 -c
> 
>                            Process Tree:
>                                a
> 
> Action: a forks b
>                                a
>                                └── b
> Action: a forks c
>                                a
>                                ├── b
>                                └── c
> Action: c EXITS
>                                a
>                                └── b
> Action: a forks d
>                                a
>                                ├── b
>                                └── d
> Action: a forks e
>                                a
>                                ├── b
>                                ├── d
>                                └── e
> ~~~

> [!question]- 2\. One control the simulator gives you is the `fork_percentage`, controlled by the `-f` flag. The higher it is, the more likely the next action is a fork; the lower it is, the more likely the action is an exit. Run the simulator with a large number of actions (e.g., `-a 100`) and vary the `fork_percentage` from 0.1 to 0.9. What do you think the resulting final process trees will look like as the percentage changes? Check your answer with `-c`.
> 挺没意思的，显然如果`fork_percentage`是最小的，最后肯定只剩一个进程a；如果是最大，那么这棵树上所有的进程都会保存下来，因为一直在fork。

> [!question]- 3\. Now, switch the output by using the -t flag (e.g., run `./fork.py -t`). Given a set of process trees, can you tell which actions were taken?
> 不回答了，太简单了。

> [!question]- 4\. One interesting thing to note is what happens when a child exits; what happens to its children in the process tree? To study this, let’s create a specific example: `./fork.py -A a+b,b+c,c+d,c+e,c-`. This example has process ’a’ create ’b’, which in turn creates ’c’, which then creates ’d’ and ’e’. However, then, ’c’ exits. What do you think the process tree should like after the exit? What if you use the -R flag? Learn more about what happens to orphaned processes on your own to add more context.
> 根据之前所说：[[#^f8d31e]]，结果很容易了。

。。。剩下的就先算了吧

#### 2.2.6.2 Code

> [!question]- 1\. Write a program that calls fork(). Before calling fork(), have the main process access a variable (e.g., x) and set its value to something (e.g., 100). What value is the variable in the child process? What happens to the variable when both the child and parent change the value of x?
> 
> ~~~c
> #include <stdio.h>
> #include <stdlib.h>
> #include <unistd.h>
> #include <sys/wait.h>
> 
> int main() {
>     int x = 100;
>     int pid = fork();
>     if (pid == 0) {
>         // child
>         printf("I am child: %d, x is %d\n", (int) getpid(), x);
>         x = 99;
>         printf("[%d] after changed x, it is %d\n", (int) getpid(), x);
>     } else {
>         wait(NULL);
>         printf("I am parent: %d, x is %d\n", (int) getpid(), x);
>         printf("[%d] after child changed x, it is %d\n", (int) getpid(), x);
>         x = 98;
>         printf("[%d] after changed x, it is %d\n", (int) getpid(), x);
>     }
>     return 0;
> }
> ~~~
> 
> ~~~shell
> ❯ ./value
> I am child: 15596, x is 100
> [15596] after changed x, it is 99
> I am parent: 15595, x is 100
> [15595] after child changed x, it is 100
> [15595] after changed x, it is 98
> ~~~
> 
> 也就是说，父进程和子进程里的x完全是独立的，怎么修改也不会相互影响。

> [!question]- 2\. Write a program that opens a file (with the open() system call) and then calls fork() to create a new process. Can both the child and parent access the file descriptor returned by open()? What happens when they are writing to the file concurrently, i.e., at the same time?
> 
> ~~~c
> #include <stdio.h>
> #include <stdlib.h>
> #include <unistd.h>
> #include <fcntl.h>
> #include <sys/stat.h>
> 
> int main() {
>     int fd = open("./temp.txt", O_WRONLY|O_CREAT|O_TRUNC, S_IRWXU);
>     if (fd < 0) {
>         perror("open");
>         exit(1);
>     }
>     int pid = fork();
>     if (pid == 0) {
>         // child
>         write(fd, "Hello from child\n", 17);
>         printf("Child wrote to the file\n");
>     } else {
>         write(fd, "Hello from parent\n", 18);
>         printf("Parent wrote to the file\n");
>     }
>     close(fd);
>     return 0;
> }
> ~~~
> 
> `temp.txt`的文件很大可能是：
> 
> ```txt
> Hello from parent
> Hello from child
> ```
> 
> 因为很大概率上父亲会先执行。但是如果由于并发，可能会导致结果很不一样，比如这些东西都混到了一起。

^e2b4a6

> [!question]- 3\. Write another program using fork(). The child process should print “hello”; the parent process should print “goodbye”. You should try to ensure that the child process always prints first; can you do this without calling wait() in the parent?
> 我看有人用sleep：[OSTEP-Homework/C5-Process-API at main · MarekZhang/OSTEP-Homework](https://github.com/MarekZhang/OSTEP-Homework/tree/main/C5-Process-API)。但我觉得这个不靠谱。因为这也不能完全保证hello先打出来。比如儿子在打印前因为OS的bug整个阻塞住了，这个时候父亲却恰巧打印出来。虽然这种情况几乎不可能发生，但是也不能完全避免。GPT给的答案是使用管道，这个比较靠谱一点：
> 
> ~~~c
> #include <stdio.h>
> #include <stdlib.h>
> #include <unistd.h>
> #include <string.h>
> 
> int main() {
>     int pipefd[2];
>     int pid;
>     char buf;
> 
>     // 创建管道
>     if (pipe(pipefd) == -1) {
>         perror("pipe");
>         exit(EXIT_FAILURE);
>     }
> 
>     pid = fork();
>     if (pid == -1) {
>         perror("fork");
>         exit(EXIT_FAILURE);
>     }
> 
>     if (pid == 0) {
>         // 子进程
>         close(pipefd[0]); // 关闭未使用的读端
>         printf("hello\n");
>         write(pipefd[1], "c", 1); // 向管道写入一个字符，通知父进程
>         close(pipefd[1]); // 关闭写端
>         exit(EXIT_SUCCESS);
>     } else {
>         // 父进程
>         close(pipefd[1]); // 关闭未使用的写端
>         read(pipefd[0], &buf, 1); // 等待从子进程读取数据
>         printf("goodbye\n");
>         close(pipefd[0]); // 关闭读端
>     }
> 
>     return 0;
> }
> ~~~

> [!question]- 4\. Write a program that calls fork() and then calls some form of exec() to run the program /bin/ls. See if you can try all of the variants of exec(), including (on Linux) execl(), execle(), execlp(), execv(), execvp(), and execvpe(). Why do you think there are so many variants of the same basic call?
> [OSTEP-Homework/C5-Process-API at main · MarekZhang/OSTEP-Homework](https://github.com/MarekZhang/OSTEP-Homework/tree/main/C5-Process-API)
> 
> 本质上就是环境变量，以及接受参数的方式不同而已。[exec (system call) - Wikipedia](https://en.wikipedia.org/wiki/Exec_(system_call)#C_language_prototypes)

> [!question]- 5\. Now write a program that uses wait() to wait for the child process to finish in the parent. What does wait() return? What happens if you use wait() in the child?
> wait会返回终止的进程的id：[wait(2) - Linux manual page](https://man7.org/linux/man-pages/man2/waitpid.2.html#RETURN_VALUE)；如果你在子进程里调用wait，并且子进程也没有子进程了，那么会立刻返回。

> [!question]- 6\. Write a slight modification of the previous program, this time using waitpid() instead of wait(). When would waitpid() be useful?
> 看[这里](https://man7.org/linux/man-pages/man2/waitpid.2.html#DESCRIPTION)：wait的返回条件是任意一个子进程结束。因此如果是多个子进程的话，waitpid就有很大用处了。

> [!question]- 7\. Write a program that creates a child process, and then in the child closes standard output (STDOUT FILENO). What happens if the child calls printf() to print some output after closing the descriptor?
> 前面提到过：[[#^9219fd]]，这里我们没有再创建文件。所以这个消息会被丢弃，不会显示在终端上。

> [!question]- 8\. Write a program that creates two children, and connects the standard output of one to the standard input of the other, using the pipe() system call.
> 
> ~~~c
> #include <stdio.h>
> #include <stdlib.h>
> #include <unistd.h>
> #include <sys/wait.h>
> 
> int main() {
>     int pipe_fd[2];
>     int pid1, pid2;
> 
>     // 创建管道
>     if (pipe(pipe_fd) == -1) {
>         perror("pipe failed");
>         exit(1);
>     }
> 
>     // 创建第一个子进程
>     pid1 = fork();
>     if (pid1 < 0) {
>         perror("fork failed");
>         exit(1);
>     } else if (pid1 == 0) {
>         close(pipe_fd[0]); // 关闭pid1的读端，因为用不到
>         write(pipe_fd[1], "Hello from child 1\n", 19);
>         close(pipe_fd[1]);
>         exit(0);
>     }
> 
>     // 创建第二个子进程
>     pid2 = fork();
>     if (pid2 < 0) {
>         perror("fork failed");
>         exit(1);
>     } else if (pid2 == 0) {
>         close(pipe_fd[1]); // 关闭pid2的写端，因为用不到
>         char buffer[128];
>         if (read(pipe_fd[0], buffer, sizeof(buffer)) > 0) {
>             printf("Child 2 received: %s", buffer);
>         }
>         close(pipe_fd[0]);
>         exit(0);
>     }
> 
> 	// 关闭父进程的管道。不能提前到进程创建之前，
>     // 因为那样的话进程还没出生，管道就没了。
>     close(pipe_fd[0]);
>     close(pipe_fd[1]);
>     
>     wait(NULL); // 等待第一个子进程结束
>     wait(NULL); // 等待第二个子进程结束
>     return 0;
> }
> ~~~
> 
> 几个要点要说：
> 
> 1. 从pipe读的时候，如果pipe没有被关闭，那么程序就会阻塞直到pipe里有东西。所以pid2中如果把if换成while，就会阻塞下去；
> 2. `pipe_fd`是个int数组，两个元素，分别是管道的读端和写端的fd。在上面的例子中，一共有几个fd？答案是6个！因为父进程，pid1和pid2都有一个`pipe_fd`数组。不过这些fd最终指向的文件是一样的，都是实际上管道的两个端（这就像c中有6个指针指向2段内存，java里6个引用指向2个对象一样）。参见[[#^e2b4a6|问题2]]，**管道也是文件**，所以不同进程的fd指向的也是同一个管道的同一个端；
> 3. 正因为2，所以才会关闭pid1的读端和pid2的写端。我们有3个fd指向读端，3个fd指向写端。只有这些东西全部被关闭了，管道才会释放。所以那些没用的东西我们完全可以关闭。
> 
> 其他人写的答案：[OSTEP-Homework/C5-Process-API at main · MarekZhang/OSTEP-Homework](https://github.com/MarekZhang/OSTEP-Homework/tree/main/C5-Process-API)。我认为这个也有问题，因为我认为从题目描述来看，三个进程的关系应该是：
> 
> ```mermaid
> graph TD
>     parent["Parent"]
>     pid1["pid1"]
>     pid2["pid2"]
>     
>     parent --> pid1
>     parent --> pid2
> ```
> 
> 而不是
> 
> ```mermaid
> graph TD
>     parent["Parent"]
>     pid1["pid1"]
>     pid2["pid2"]
>     
>     parent --> pid1
>     pid1 --> pid2
> ```