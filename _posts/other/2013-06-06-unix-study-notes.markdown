---
layout: post
title:  "Unix初级编程学习笔记"
date:   2013-06-06 19:12:03
categories: Unix
type: other
---

> 本手记主要记录理解unix编程的内容

**第二章内容总结**  

**1.**Unix并不提供恢复删除文件的功能，原因是Unix是一个多用户的系统，当一个文件删除以后，它所占用的空间就会立即分配给其他用户的文件  

**2.**事实上，编写Unix程序所需的资料都在系统中，我们需要知道的仅仅是如何找到这些资料而已  
--阅读联机帮助，man  
--搜索联机帮助, man -k | grep 'keyword'  
--阅读.h文件  
--从参阅部分（SEE ALSO）得到启示  

**3.**操作文件系统调用与头文件  
{% highlight c %}
  open(filename, how);  <fcntl.h>
  creat(filename,mode); <fcntl.h>
  read(fd,buf,size);    <unistd.h>
  close(fd);		<unistd.h>
  write(fd, buf,size);	<unistd.h>
  lseek(fd,dist,base);	<sys/types.h> <unistd.h>
  exit(number);		<stdlib.h>
  perror(string);	<errno.h>
  time(t);		<time.h> man -k you will get a lot
  ctime(t);		<time.h> man -k
  get utmp file;	<utmp.h>
  getuid(),geteuid();<unistd.h> <sys/types.h>//获取用户uid  
  getpw(),getpwuid()//等，可以获取passwd文件的信息，返回一个结构体，<pwd.h>,<sys/types.h>
{% endhighlight %}

**4.**文件描述符表示文件和进程之间的连接,shell使用文件描述符0与进程的边准输入相关联，1与标准输出相关联，2与标准错误输出相关联，对应的常量为STDIN_FILENO,STDOUT_FILENO,STDERR_FILENO,定义在unistd.h  

**5.**当系统调用出错时会把全局变量errno的值设为相应的错误代码，然后返回-1（一般错误都返回这个），程序可以通过检查errno来确定错误的类型，并采取相应的措施。  

**第三章内容总结**  

**1.**文件许可权限的数字是八进制，一共16位  
**2.**子域掩码是系统编程中一种重要且常用的技术  
**3.**SUID位在用户的执行权限x位置，用s表示。SUID位告诉内核，运行这个程序的时候认为是由文件所有者在运行这个程序，但是这个程序通过getuid可以知道真正谁在运行这个程序。  
**4.**SGID位在组的执行权限x位置，用s表示。如果一个程序所属的组设置为g，而且它的set-group-ID位也被设置，那么程序运行的时候就好像它正被g组中的某一个用户运行一样。  
**5.**sticky位在Other的执行权限x位置，用t表示。现在只能作用与目录，使得目录里面的文件只能被创建者删除  
**6.**操作文件属性的系统调用与头文件  
{% highlight c %}
  opendir(dirname)  	<dirent.h>
  readdir(dir_ptr)  	<dirent.h>
  stat(filename,bufp)	<sys/stat.h>
  getpwuid(uid)		<pwd.h>
  getgrgid(gid)		<grp.h>
  strcpy(str1,str2)	<string.h>
  gmtime(time_t)	<time.h>
  chmod(path,mode)	<sys/types.h> <sys/stat.h>
  chown(path,owner,group)	<unistd.h>
  utime(path,utimbuf,newtimes)	<sys/time.h> <utime.h> <sys/types.h>
  rename(old,new)	<stdio.h>
{% endhighlight %}

**7.**关于数组与指针  
声明数组时，同时分配了一些内存空间，用于容纳数组元素，但是声明指针时，只分配了用于容纳指针本身的内存空间  
规则1\.表达式中的数组名被编译器当作一个指向该数组的一个元素的指针  
规则2\.下表总是与指针的偏移量相同  
规则3\.在函数的声明中（形式参数），数组名被编译器当作指向该数组第一个元素的指针  

**第四章内容总结**  
**1.**在Unix的目录结构中，这两个链接的状态完全相同；他们被称为指向文件的硬链接。文件是一个i-node和一些数据块block的结合；链接是对i-node的引用。  
**2.**ino_t在/usr/include/linux/coda.h定义，是unsigned long类型  
**3.**Unix上的每个运行程序都有一个当前目录，chdir系统调用改变进程的当前目录。  
**4.**一个i-node节点号并不唯一地标识一个文件。因为把不同的文件系统挂载后可以有相同的i-node号。  
**5.**硬链接是将目录链接树的指针，硬链接同时也是将文件名和文件本身链接起来的指针。符号连接通过名字（路径）引用文件，而不是i-node号。  
**6.**操作目录的系统调用与头文件
{% highlight c %}
  mkdir(pathname,mode)  <sys/stat.h> <sys/types>
  rmdir(path) 		<unistd.h>
  unlink(path) 		<unistd.h>
  link(orig, new)	<unistd.h>
  rename(from, to)	<unistd.h>
  chdir(path)		<unistd.h>
{% endhighlight %}

**第五章内容总结**  
**1.**设备文件是链接，而不是容器。设备文件的inode存储的是指向内核子程序的指针，而不是文件的大小和存储列表。内核中传输数据的子程序被称为设备驱动程序。  
**2.**在/dev/pts/2这个例子中，从终端进行数据传输的代码是在设备-进程表中编号为136的子程序。该子程序接受一个整型参数。在/dev/pts/2中，参数是2.136和2这两个数被称为设备的主设备号和从设备号。主设备确定处理该设备实际的子程序。而从设备号被作为参数传输到该子程序。  
**3.**设备文件的inode节点包含指向内核子程序表的指针。主设备号用于告知从设备读取数据的那部分代码位置。  
**4.**当文件的描述符的O_APPEND位被开启后，每个对write的调用自动调用lseek将内容添加到文件的末尾。  
**5.**对于lseek和write的调用是独立的系统调用，内核可以随时打断进程，从而使后面的这两个操作被中断。当O_APPEND被设置后，内核将lseek和write组合成一个原子操作，被连接一个不可分割的单元。  
**6.**stty -echo关闭回显。stty erase X设置X为删除键。 一次行改变多个设置stty erase @ echo  
**7.**操作连接控制的系统调用与头文件  
{% highlight c %}
  fcntl(fd, cmd)	<fcntl.h> <unistd.h> <sys/types.h>
  tcgetattr(fd, info)	<termios.h> <unistd.h>
  tcsetattr(fd, when, info)	<termios.h> <unistd.h>
  ioctl(fd, operation..)	<sys/ioctl.h>
{% endhighlight %}
  
**第六章内容总结**  
**1.**缓冲、回显、编辑和控制键处理都是由驱动程序完成的。  
**2.**stty -icanon关闭驱动程序中的规范模式处理，即没有了缓冲和编辑处理功能。  
**3.**当调用getchar或read从文件描述符读取输入时，这些调用通常都会等待输入，即阻塞输入，一直等待用户输入，直到用户输入一个字符或是检测到文件的末尾。  
阻塞不仅仅是终端连接的属性，而是任何一个打开的文件的属性。程序可以使用fcntl或是open为文件描述符启动非阻塞输入，开启O_NDELAY标志。  
关闭一个文件描述符的阻塞状态并调用read，如果能够获得输入，read获得输入并返回所获得的字符个数。如果没有输入字符，read返回0，这就像遇到文件末尾一样，如果有错误，read返回-1。getchar非阻塞输入，如果能够获得输入，getchar获得输入，如果没有输入字符，getchar返回EOF。  
**4.**终端驱动程序不仅一行行地缓冲输入，而且还一行行地缓冲输出。驱动程序缓冲输出，直到它收到一个换行符或者程序试图从终端读取输入。  
**5.**SIGKILL和SIGSTOP两个信号不可被阻挡  
**6.**信号是从内核发送给进程的一种短简消息。信号可能来自用户、其他进程或内核本身。进程可以告诉内核，在它收到信号时需要做出什么样的回应。  
**7.**操作信号的系统调用：
{% highlight c %}
  signal(signum, void(*action)(int)) <signal.h>
{% endhighlight %}

**第七章内容总结**  
**1.**一个控制终端屏幕的库,curses库， 包含curses\.h 编译时链接 -l curses。curses通过虚拟屏幕来最小化数据流量。  
真实屏幕是眼前的一个字符数组。curses保留屏幕的两个内部版本。一个内部屏幕是真实屏幕的复制。另一个是工作屏幕，其上记录了对屏幕的改动。  
**2.**sleep(n)将当前进程刮起n秒或者在此期间被一个不能忽略的信号的到达所唤醒。  
**3.**系统中的每个进程都有一个私有的闹钟。这个闹钟很像一个计时器，可以设置一定秒数后闹铃。时间一到，时钟就发送一个信号  
SIGLARM到进程。除非进程为SIGLARM设置了处理函数（handler），否则信号将杀死这个进程。调用alarm(0)意味着关闭闹钟。  
**4.**系统内有三种计时器：真实、进程和使用。每个计数器有两个参数：初始时间和重复间隔  
**5.**unix中的软件中断被称为信号signal。信号处理函数每次使用后被禁用称做捕鼠器模型。  
**6.**释放资源时恢复获取时的状态是个好习惯。  
**7.**一个信号处理者或者一个函数，如果在激活状态下能被调用而不引起任何问题就称之为可重入的。  
**8.**异步读入的优势在于程序不用被输入阻塞而可以做些其他什么。  
**9.**操作系统的系统调用
{% highlight c %}
  sleep(seconds)  <unistd.h>
  alarm(seconds)  <unistd.h>
  pause()	  <unistd.h>
  getitimer(which, timeval)	<sys/time.h>
  setitimer(which, timeval, oldtimeval)	<sys/time.h>
  sigaction(signum, action, prevaction) <signal.h>
  sigprocmask(how, sigs, prev)	<signal.h>
  kill(pid, sig)	<sys/types.h> <signal.h>
{% endhighlight %}

**第八章内容总结**  
**1.**unix系统的文件系统把一对旋转圆盘上连续的簇编程有序组织的树状目录结构，它的进程系统将硅片上的一些位组织成一个进程社会--成长、互相影响、合作、出生、工作和死亡。  
**2.**execvp是一组基于execve系统调用函数中的一个，它们统称为exec。exevcvp参数的数组第一个元素设置为程序的名称，最后一个参数必须为null。  
**3.**exec系统调用从当前进程中把当前程序的机器指令清除，然后在空的进程中载入调用时指定的程序代码，最后运行这个新的程序。exec调整进程的内存分配使之适应新的程序对内存的要求。  
**4.**fork:(1)分配新的内存块和内核数据结构。（2）复制原来的进程到新的进程.（3）向运行进程集添加新的进程。（4）将控制返回给两个进程  
**5.**wait暂停调用它的进程直到子进程结束。然后，wait取得子进程结束时传给exit的值  
wait阻塞调用它的程序直到子程序结束。wait返回结束进程的PID  
父进程调用wait时传一个整型变量地址给函数，内核将子进程的退出状态保存在这个变量中。  
**6.**按照Unix惯例，成功的程序调用exit(0)或者从main函数中return 0.程序遇到问题而退出调用exit时传给它一个非零值。  
**7.**键盘信号发送给所有连接到终端的进程。  
**8.**用进程编程，execvp/exit就像call/return  
**9.**全局变量是有害的，它破坏了封装原则，导致出人意料的副作用和难以维护的代码。  
**10.**exit刷新所有的流，调用由atexit和on_exit注册的函数，执行当前系统定义的其他与exit相关的操作。然后调用_exit。  
**11.**操作进程的系统调用
{% highlight c %}
  execvp(file, argv)	<unistd.h>
  fork(void)	<unistd.h>
  wait(status)	<sys/types.h>	<sys/wait.h>
  _exit(status)	<unistd.h>	<stdlib.h>
{% endhighlight %}


**第九章内容总结**  
**1.**在shell中，条件是一个命令，返回正值意味着命令执行成功。  
**2.**if语句还有另一特征。如果if后的条件是一系列的命令，那么最后一个命令的exit值被用作这个语句块的条件值，并由此来决定条件是否成立。  
**3.**set是shell的一个命令，而不是一个由shell运行的程序。  
**4.**Unix允许用户在称之为环境的地方以变量的形式存放设置。  
**5.**环境是每个程序都可以存取的一个字符串数组，数组的地址被存放在一个名为environ的全局变量里。  
**6.**子程序中的环境的设置是父进程环境的副本，子进程不能修改父进程的环境。因为在进程调用fork和exec时整个环境都被自动复制了，所以通过环境来传递数据比较方便，快捷。  
**7.**启动进程以&符号结尾，所以shell将运行它但是不调用wait。  
**8.**系统调用
{% highlight c %}
  isdigit(c)	<ctypes.h>
  isalnum(c)	<ctypes.h>
{% endhighlight %}


**第十章内容总结**  
**1.**所有的UNIX工具都使用文件描述符0，1和2  
**2.**通常通过shell命令运行Unix工时，stdin,stdout,stderr连接到终端上。通过使用输出重定向标志,命令cmd>filename告诉shell将文件描述符1定位到文件。  
**3.**文件描述符的概念非常简单：它是一个数组的索引号。每个进程都有其打开的一组文件。这些打开的文件被保持在一个数组中。文件描述符即为 某文件在此数组的索引。  
**4.**当打开文件时，为此文件安排的描述符总是此数组中最低可用位置的索引。  
**5.**运行exec之后，进程的用户ID不会改变，其优先级不会改变，并且其文件描述符也是和运行exec之前一样。shell使用进程通过fork产生子进程与子进程调用exec之间的时间间隔来重定向标准输入、输出到文件。  
**6.**array[0]为读数据端的文件描述符，而array[1]则为写数据端的文件描述符。像打开一个文件的内部情况一样，管道的内部实现隐藏在内核中，进程只能看见来两个文件描述符。类似于open调用，pipe调用也使用最低可用文件描述符。  
**7.**从本质上讲，管道是一个特殊的文件。管道是以Unix文件系统为基础而设计的，它具有对普通文件操作的一般特性，如用于普通文件的打开（OPEN）、读（READ）、写（WRITE）、关闭（CLOSE）等系统调用同样可以作用于管道。  
**8.**管道与文件是不一样的,它是一个数据队列：管道读取阻塞，多个读者可能会引起麻烦，写入数据阻塞知道管道有空间去容纳新的数据，写入必须保证一个最小的块，若无读者在读取数据，则写操作执行失败。  
**9.**系统调用
{% highlight c %}
  dup(fd)	<fcntl.h>
  dup2(fd,close_fd)	<fcntl.h>
  pipe(array[2])	<unistd.h>
{% endhighlight %}


**第十一章内容总结**  
**1.**在while语句判断条件表达式中可以写多个语句，用逗号分开，判断以最后一个语句为条件，请看tinybc.c  
**2.**dc是逐行处理数据和命令的。  
**3.**一个客户/服务器模型程序要成为协同系统必须有明确指明消息结束的方法，并且程序必须使用简单并可预测的请求和应答。  
**4.**popen运行了一个程序并返回指向该程序标准输入或标准输出的连接。  
**5.**sh支持-c选项，用以告诉shell执行某命令后退出。  
**6.**管道在一个进程中被创建，通过fork来实现共享。因此，管道只能连接相关的进程，也只能连接同一台主机上的进程。  
**7.**编写网络服务端步骤：socket->bind->listen->accept->read/write->close  
**8.**地址就是一个以主机和端口为成员的结构体，自己写的程序应该首先初始化该结构体的成员，然后再具体填充主机地址和端口后，最后填充地址族。  
**9.**socket文件描述符实际上是连接到呼叫进程的某个文件描述符的一个连接。  
**10.**编写网络客户端步骤：socket->connect->read/write->close  
**11.**客户端必须建立Internet域（AF_INET）socket，并且它还必须是流socket（SOCK_STREAM）。  
**12.**popen系统调用对于编写网络服务来说是危险的，因为它直接把一行字符串传给shell。在网络程序中，将字符串传给shell是一个非常错误的想法。  
**13.**系统调用
{% highlight c %}
  execlp(file, args...)	<unistd.h>
  fdopen(fd, mode)	<stdio.h>
  sscanf(source, format, ...)	<stdio.h>
  popen(command, type)	<stdio.h> 
  bzero(s, size)	<strings.h>
  gethostname(hostname, len)	<unistd.h>
  gethostbyname(hostname)	<netdb.h>	
  bcopy(src,dest,size)	<strings.h>
  htons(value)		<netinet/in.h>
  socket(domain, type, protocol)	<sys/types.h> <sys/socket.h>
  bind(sockid, addrp, addrlen)	<sys/types.h> <sys/socket.h>
  listen(sockid, qsize)	<sys/socket.h>
  accept(sockid, callerid, addrlenp)	<sys/types.h> <sys/socket.h>
  atoi(nptr)	<stdlib.h>
  connect(sockid, serv_addrp, addrlen)	<sys/types.h> <sys/socket.h>
{% endhighlight %}

**第十二章内容总结**  
**1.**服务器设计的两种方法：a.服务器接收请求，自己处理，用于快速简单的任务。b.服务器接收请求，然后创建一个新进程来处理工作，用于慢速的更加复杂的任务  
**2.**有的服务器容易配置和操作，有的提供了更多的安全特征，有的则快速处理请求或使用较少的内存。  
**3.**Web服务器处理3种类型的请求：返回文件内容、目录列表和运行程序。  
**4.**waitpid接口提供了wait函数超集的功能，可以处理信号，内部实现要去看内核才知道。  
**5.**系统接口：
{% highlight c %}
  waitpid(pid, status, options)	<sys/types.h> <wait.h>
{% endhighlight %}


**第十三章内容总结**  
**1.**一种实施许可证技术是编写程序来执行许可证控制。通常的做法是设计一个许可证服务器获得许可的应用程序。该服务器是一个进程，他授权应用程序运行。  
**2.**本地地址通常叫Unix域地址，它是一个文件名（例如/dev/log, /dev/printer, /tmp/lserversock）,没有主机和端口号。  
**3.**系统调用接口  
{% highlight c %}
  sento(socket, msg, len, flags, dest, dest_len)	<sys/types.h> <sys/socket.h>
  recvfrom(socket, msg, len, flags, sender, sender_len)	<sys/types.h> <sys/socket.h>
  inet_ntoa(addr)	<sys/socket.h> <netinet/in.h> <arpa/inet.h>
  ntohs(value)	<netinet/in.h>
{% endhighlight %}
**4.**
  socket 		PF_INET			PF_UNIX  
  SOCK_STREAM		连接的，跨机器的	连接的，本地的  
  SOCK_DGRAM		数据报，跨机器的	数据报，本地的  


**第十四章内容总结**  
**1.**pthread_join使得调用线程挂起直至由thread参数指定的线程终止。  
**2.**多个线程在一个单独的进程进行，共享全局变量，因此线程间可以通过设置和读取这些全局变量来进行通信。  
**3.**由于对设置释放互斥锁太多的调用导致了系统性能的降低。  
**4.**每个进程有其独立的数据空间、文件描述符以及进程的ID。而线程共享一个数据空间、文件描述符以及进程ID。  
**5.**某个线程中的函数调用了fork，则只有调用fork的线程会在新的进程中运行，不会复制其他线程。  
**6.**pthread_cond_wait先自动释放指定的锁，然后等待条件变量的变化，如果在调用此函数之前，互斥量mutex没有被锁住，函数的执行结果是不确定的。在返回原调用函数之前，此函数自动将指定的互斥量重新锁住。  
**7.**系统调用接口
{% highlight c %}
  pthread_create(thread, attr, func, arg)	<pthread.h>
  pthread_join(thread, retval)			<pthread.h>
  pthread_mutex_lock(mutex)			<pthread.h>
  pthread_mutex_unlock(mutex)			<pthread.h>
  pthread_cond_wait(cond, mutex)		<pthread.h>
  pthread_cond_signal(cond)			<pthread.h>
{% endhighlight %}


第十五章内容总结
**1.**共享内存有自己的拥有者并且为用户、组或者其他成员设置了权限位，来控制他们各自的访问权限。  
**2.**信号量是一个内核变量，他可以被系统中的任何进程所访问。进程间可以使用这个变量来协调对于共享内存和其他资源的访问。  
**3.**学习Unix系统编程的最好的办法就是不断的读程序，写程序。大家多注意一下每天都使用的程序，还有一些吸引你的程序。通过使用，学习，并且经常自己来实现一些已经存在的程序，就可以更深，更精，更广泛地了解Unix的编程了。  
**4.**系统调用接口  
{% highlight c %}
  select(numfds, read_set,write_set,error_set,timeout)	<sys/time.h>
  shmget(key, size-of-segment, flags)			<sys/shm.h>
  shmat(seg_id, NULL,flags)				<sys/shm.h>
  semget(key, numsems, flags)				<sys/sem.h>
  semctl(semset_id semnum, cmd, arg)			<sys/sem.h>
  semop(semid, actions, numactions)			<sys/sem.h>
{% endhighlight %}
