# Linux System Programing
基本概念：
* 文件
    * 常规文件
    * 目录（硬链接表）
    * 硬链接（filename->inode）
    * 符号链接（可以跨文件系统，需要开个inode和数据块记录完整路径）
* 进程：运行中的目标代码+数据+打开的资源+相关状态
* 线程：栈+处理器状态+PC
* 用户和用户组
    * real pid/gid
    * effective pid/gid - root可以指定用什么身份
    * saved pid/gid - 修改身份后保存之前用的
    * filesystem pid/gid - 等同于effective pid/gid
* 文件系统权限 - user/group/other
* 信号
* IPC：管道、命名管道、信号量、消息队列、共享内存、futex（fast userspace mutex）
* 页面缓存：缓存从文件系统读到的数据

文件读写：
* open() / close()
* read() / write()
* fsync() - 只刷入某个fd的脏数据
* fdatasync() - 只刷入数据，不刷入元数据
* sync() - 刷入OS中的所有脏数据

I/O多路复用：
* select() - 不能指定单独监听某个FD、监听哪些事件（如果要监听999，每次都要检查完0-999）
* poll() - 可以单独指定监听某个FD的某些事件，但仍需手动遍历所有注册的FD检查事件是否发生
* epoll()
    * 需要开个FD专门作为epoll的上下文
    * 注册FD时可携带额外数据（类似Selector的attachment()）
    * 每次查询后将发生了相关事件的`epoll_event`写入给定的内存中，避免手动检查哪些FD可用

文件映射到内存：
* mmap()
* mremap()
* munmap()
* msync()
* mprotect()
* madvice()

进程管理：
* pid / ppir (parent)
* user / group
* process group
* exec*() - 执行程序，即加载文件替换当前的进程
    * 丢失待处理信号、信号回调、内存锁、mmap等信息，地址空间重置
    * 保留pid、ppid、进程优先级、user与group等属性
    * 打开的FD默认会保留（可用fcntl()控制）
* fork()
    * 丢失待处理的信号、已获得的文件锁
    * 内存页面`copy-on-write`
* exit() - 在用户态执行下列操作：
    * atexit()回调
    * 刷回所有标准I/O流
    * 移除tmpfile()创建的临时文件
* _exit() - 在内核态执行清理：
    * 回收内存
    * 关闭打开的文件
    * 销毁进程并通知父进程（SIGCHLD）
* wait() / waitpid() / waitid()
* system()
* setuid()/setgid() / seteuid()/setegid()
* getuid()/getgid() / geteuid()/getegid()

会话/进程组：
* 信号能群发给会话或进程组中的所有进程
* pgid == 进程组组长的pid
* 会话 - 一个或多个进程组（fg组+若干个bg组）
* 会话ID - 会话主进程的pid 
* setsid() / getsid()
* setpgid() / getpgid()
* 开一个daemon
    * 执行以下步骤：主进程fork()，主进程exit()，子进程setsid()，子进程chdir("/")（避免原WD所在文件系统不能umount），子进程关闭所有打开的FD，子进程打开stdin、stdout、stderr并重定向到/dev/null
    * daemon()

多线程：
* 线程模型（1-1、1-N、N-M）
* pthread_*()

文件元数据：
* inode号
* 物理位置、大小
* 所有者信息（user、group）
* 访问权限
* 相关的时间戳

目录信息：filename -> inode

内存锁：锁某个内存区域，防止换页
