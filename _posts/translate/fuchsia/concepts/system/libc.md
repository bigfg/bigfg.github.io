# libc

## libc是什么

在Posix系统上，程序链接到一个库(动态库或者静态库)成为libc。这些库提供了
标准C定义的函数，以及支持C程序的运行环境。很多系统在这些库中还定义了支
持其他特定平台的接口。这类接口是用户空间访问内核功能的首选方法。比如说，
Posix系统的libc库中，提供了一个访问系统调用open的open函数。很多时候，这
种标准是跨平台的，比如说pthreads。其他的是访问特定于内核的功能，比如说
epoll或kqueue。无论如何，这类库存在于系统本身，并且提供稳定接口。相反，
在32位稳定的windows接口里面，并不提供系统范围的libc。

Fuchsia和Posix系统有不太一样，首先，Zircon内核(Fuchsia的微内核)并不提供
标准的Posix类型的系统调用接口。因此像open这样的Posix功能并不能调用Zircon
的open系统调用。其次，Fuchsia实现了部分Posix标准，但是忽略了大部分的模型。
其中比较明显的缺失，signals，fork以及exec。第三，Fuchsia并不要求程序使用
libc的应用程序二进制接口。应用程序可以使用自己的libc，或者压根就不使用。
然而，Fuchsia确实提供了一个程序可以动态链接的libc.so，libc.so实现了C标准
库，以及实现了部分Fuchsia支持的、类似传统Posix系统的那样Posix接口。

## 实施操作

下面列举了部分在Fuchsia的libc实现的(或没有实现)功能。

### C标准库

Fuchsia的libc实现了C11标准。尤其是包含了线程相关的接口，比如threads(
thrd_t)和mutexes(mtx_t)。一小部分有用的扩展，用于将C11结构，比如thrd_t
桥接到内核对应的结构，比如zx_handle_t上。

### Posix

Posix定义了一部分接口，这些包含(不尽含)：文件IO，BSD套接字，以及pthreads。

### 文件IO和BSD Sockets

Zircon是一个微内核，并不会实现文件IO的相关操作。相反的，Fuchsia的用户
态服务提供文件系统功能。libc本身为Posix文件IO功能定义了弱符号，例如
open，write和fstat，但是对他们的调用都简单的失败。除了libc.so，程序还可
以链接到fdio.so库。fdio.so知道如何通过Channel IPC和其他用户态服务交互，
并提供类Posix层供暴露出来。套接字实现类似，通过和用户态网络功能栈进行交互
的fdio.so实现。

### pthreads

Fuchsia的libc提供了pthread的部分标准。特别是，核心部分比如pthread_t(
和C11概念直接对应的部分)，以及像pthread_mutex_t的同步原语都被提供了。
某些细节未实现，例如进程共享的互斥锁，实施的是子集，并不完全旨在全面。

### Signals

Fuchsia并没有Unix类型的信号，Zircon也没有提供实现它们(内核没有办法使
得线程跳出其当前的执行)的方式。因此，Fuchsia的libc没有提供信号安全函数
的概念，并且内部机制也没有实现信号感知的机制。

由于上面的原因，libc函数不会使用EINTR，也只有Fuchsia的代码才不需要考虑
这种情况。然而，这么做确实完全安全的。Fuchsia仍然定义EINTR常亮，并且为
Posix和Fuchsia编写的代码仍然具有处理EINTR处理循环。

### fork and exec

Zircon不支持fork和exec。相反的，进程创建有[fdio](/zircon/system/ulib/fdio)
提供。Zircon有进程和线程对象，但是比较原始，且对ELF没有任何认知。fdio_spawn
簇函数知道如何将ELF和初始化状态编程可执行进程。
