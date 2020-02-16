# Zircon Kernel Concepts

## 简介

内核管理着一组不同类型的对象。实现了Dispatcher接口的类的对象，可以通过系统调用直接访问。
它们都在kernel/object目录里面实现，它们有一些是自我包含的高级别对象，有一些则是基于低级别的
lk原语进行的二次包装。

### 系统调用

用户态代码通过系统调用和内核对象交互，并且机会只能通过Handles。在用户态，Handle是一个32位整形
类型(zx_handle_t类型)。当系统调用执行时，内核首先检查Handle参数是否对应一个存在于调用进程句柄
表里面的实际句柄。紧接着，内核Handle是否具有合适的类型(给一个需要事件句柄的系统调用传递系统句
柄会导致错误)，并且校验Handle包含请求操作需要的权限。

从访问的角度来看，系统调用包含三个大的分类：

1、没有限制的调用，这类调用只有很少几个，比如zx_clock_get()和zx_nanosleep()应该可以被所有线程
调用。

2、需要一个Handle作为第一个参数的系统调用，这个Handle用于表示调用要操作的内核对象，这部分调用
占大多数，比如zx_channel_write()和zx_port_queue()。

3、在内核创建对象的系统调用，这部分系统调用确实不需要Handle作为参数，比如zx_event_create()和
zx_channel_create()。对这些调用的访问以及限制是有调用线程所属的Job组来控制的。

系统调用通过libzircon.so提供，libzircon.so是Zircon提供给用户态的虚拟共享库，使用更加广泛的名
称是virtual Dynamic Shared Object 或者 vDSO。它们是形如zx_noun_verb()或者zx_noun_verb_direct-object()
的C ELF ABI(C可执行可链接的应用程序二进制接口)函数。

系统调用以定制化的FIDL被定义在//zircon/vdso中。这些定义首先被fidlc处理，然后kazoo从fidlc中获得
IR表示，处理后输出被用在VDSO和内核等的胶水的不同类型。

### Handles 和 Rights

对象可能有多个Handles指向它们(一个或者多个Process)。

基于几乎所有的内核对象来说，当指向它的最后一个Handle关闭时，这个内核对象要么也关闭，要么被放到
一个未完成的终态状态上。

通过写入Channel(使用zx_channel_write())，Handles可以从一个进程转移到其他的进程。还有另外一种传
递方式，在一个新的Process中，通过zx_process_start()传递Handle作为第一个线程的参数。

在一个Handle或它指向的内核对象执行的操作，由这个Handle绑定的Rights管理。绑定到同一个内核对象的
Handles可能具有不相同的Rights。

以Handle作为参数，zx_handle_duplicate()和zx_handle_replace()系统调用可以用来额外获取指向同一个
对象的Hanldes，并且可选择更小集合的权限。系统调用zx_handle_close()用来关闭Handle，如果关闭的是
对象的最后一个Handle，就释放这个对象。类似的，zx_handle_close_many()用来关闭一组Handles。

### Kernel Object IDs

内核中每一个对象都有一个内核对象ID，koid。Koid是一个64位的无类型整形，用于在整个操作系统生命周
期内唯一标识一个内核对象，这也说明了koid从不重用。

有两个特殊的koid：

ZX_KOID_INVALID值为零，用于标识null。

ZX_KOID_KERNEL用于标识唯一的存在的内核。

内核只用63(非常充足)位来生成koid，主要目的是为了让人工分配koids能够服用这个重要的位组流出足够的
空间。内核生成kiods的分配顺序是不确定的，并且可能会变化。

人工koids的存在是为了支持标识人工对象，比如供追踪工具使用的虚拟线程对象。人工koids如何分配留给
每一个程序，本文档不强作任何规则和约定。

### 消息传递：Sockets和Channels

Sockets和Channels都是双向和双端的IPC对象，创建Sockets和Channels都会返回两个Handles，每一个Handles
指向对象的一个端。

Sockets是流式对象，数据以单或做字节为单位写入或者从中读出。Short writes(Sockets缓存满了)和Short
reads(需要读的数据比读缓存要多)都是可能存在的。

Channels是面向数据报文的，有一个最大的由ZX_CHANNEL_MAX_MSG_BYTES标识的最大报文长度，以及对多支持
ZX_CHANNEL_MAX_MSG_HANDLES个Handles绑定到消息上。Channels并不支持short reads或者writes，消息要么
合适要么不合适。

当Handles被写入到Channel中，它们就被从发送Process中移除了。当一个包含Handles的消息被读出后，Handles
就被增加到读取的Process中。在这两个事件发生之间，Handles持续存在(以保证它们指向的对象能够存活)，
除非它们被写向的Channel的端被关闭，此时所有在途的消息都丢弃，当然所有它们包含的Handles都关闭。

参考：zx_channel_create()，zx_channel_read()，zx_channel_write()，zx_channel_call()，zx_socket_create()，
zx_socket_read()，以及zx_socket_write()。

### Objects and Signals

对象最多有32个信号(zx_signals_t类型和ZX_SIGNAL定义)，代表了当前进程状态的一些信息。Channels和Sockets
可能READABLE或WRITABLE，Processes或Threads可以TERMINATED，等等。

线程有可能在等一个或者多个对象的信息，以变成活跃。

参考signals获得更多的信息。

### Waiting：Wait One，Wait Many，and Ports

一个线程可以通过zx_object_wait_one()等待一个句柄的信号以激活，或者zx_object_wait_many()等待多个
句柄的信号以激活。两个方法，都支持超时，没有信号的情况下也会返回。

超时时间可能因为计时器松懈而偏离设定的期限，参考timer slack已获得更多信息。

如果线程要等待大量的句柄，更高效的方式是使用Port，一个其他对象可以绑定上的对象，，并且当它们声称
信号的时候，Port可以收到关于待处理信号的信息的数据包。

参考：zx_port_create()，zx_port_queue()，zx_port_wait()，和zx_port_cancel()。

### Events，Event Pairs

Event是最简单的对象，除了其活动信号外不包含任何其他信号。

Event Pair是一组发送信号的Event，Event Pair很大的一个用途，即当一个Event关闭(所有指向它的
Handles都关闭了)，PEER_CLOSED信号就在另外一侧生效。

参考：zx_event_create()，和zx_eventpaire_create()。

### Shared Memory：Virtual Memory Objects(VMOS)

虚拟内存对象代表一组物理内存页面，或者潜在页面(created/filled lazilly，按需)。

它们可以通过zx_vmar_map()被映射到进程的地址空间，通过zx_vmar_unmap()解映射。映射页面的权限可以
通过zx_vmar_protect()进行调整。

通过zx_vmo_read()和zx_vmo_write()对VMOs直接进行读和写。因此，对于一次性操作而言，地址映射的开销就
可以被避免，比如"创建一个VMO，写一个数据集到里面，并把它交个另一个进程使用"。

### Address Space Management

虚拟内存地址区域(VMARs)提供了管理一个进程地址空间的抽象。地址创建的时候，一个指向根VMAR的handle被
传递给这个进程创建者。这个句柄指向了包含整个地址空间的VMAR，这个空间可以通过zx_vmar_map()和zx_vmar_allocate()
雕刻出来，zx_vmar_allocate()可以用来生成新的VMARS(子区域或者孩子)，被用来组成地址空间。

参考：zx_vmar_map()，zx_vmar_allocate()，zx_vmar_protect()，zx_vmar_unmap()，和zx_vmar_destory()。

### Futexes

Futexed是内核原语，配合用户态的原语操作，以实现高效的同步原语。比如，Mutexes在有竞争的情况下，只需要
一个系统调用。通常情况下，它们只对标准库的实现才有意义。Zircon的libc和libc++标准库提供了针对Futexes实
现的C11，C++以及pthread类型的API。

参考：zx_futex_wait()，zx_futex_wake()，和zx_futex_requeue()。
