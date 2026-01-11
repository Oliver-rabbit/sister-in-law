# IO基础
## 系统调用
### 系统调用的定义
作系统的主要功能是为管理硬件资源和为应用程序开发人员提供良好的环境来使应用程序具有良好的兼容性。为了达到这个目的，内核提供一系列具备预定功能的函数接口即系统调用，呈现给用户。它是应用程序与操作系统间的接口与纽带，当用户访问系统调用，系统调用把应用程序的请求传递给内核，调用相应的内核函数完成所需处理，将处理结果返回给应用程序。
在没有任何库的支持下，原始的系统调用接口格式：
```c
_syscallX(type,name,type1,arg1,type2,arg2,...)
```
### 系统调用的优点
1. 为用户空间进程提供了访问内核的接口
2. 把用户从底层的硬件编程中解放出来：而无需了解底层设备的具体操作技术
3. 极大的提高了系统的安全性：所有对硬件设备资源的访问都需要通过系统调用，保护系统稳定可靠，防止应用程序恣意妄为。
### 系统调用的不足
1. 不同操作系统系统调用不兼容，程序移植工作量大。
2. 系统调用接口功能简单单一（每个系统调用都应该有一个明确的用途，Linux中不提倡采用多用途的系统调用）无法满足复杂程序的要求。

# 文件IO
## 文件的分类

| 类型     | 符号标识 | 描述                           |
| :----- | :--: | :--------------------------- |
| 普通文件   |  -   | 包含文本、二进制数据、图片、视频等            |
| 目录文件   |  d   | 存储文件名和文件信息的文件，可以包含其他文件和子目录   |
| 符号链接文件 |  l   | 指向另一个文件或目录的快捷方式              |
| 字符设备文件 |  c   | 提供串行访问的设备，数据以字符流形式传输（如键盘、终端） |
| 块设备文件  |  b   | 提供随机访问的设备，数据以块形式传输（如硬盘、U盘）   |
| 管道文件   |  p   | 用于进程间通信（IPC），一个进程写入，另一个进程读取  |
| 套接字文件  |  s   | 用于网络通信或进程间通信，支持双向数据流         |
## 文件描述符
文件描述符（File Descriptor，简称fd）是一个非负整数，是操作系统内核用来标识和管理已打开文件的抽象概念。是进程级的，每个进程有自己独立的文件描述符表。它是一个索引，指向内核维护的系统级文件表。
### 文件描述符简介
1. 文件描述符是文件IO的的操作对象
2. 文件描述符是一非负的整数，内核以此来标识一个特定进程已打开的文件。每当打开一个现存文件或创建一个新文件时，内核将向进程返回一个文件描述符，当对文件进行相应操作的时候，使用文件描述符作为参数传递给相应的函数
3. 文件描述符分配原则：<mark style="background: #FF5582A6;">顺序分配，最小可用</mark>
4. 通常一个进程启动时，都会打开三个文件：标准输入、标准输出、标准错误输出，这三个文件的文件描述符分别是0、1、2，对应的宏定义是STDIN_FILENO、STDOUT_FILENO、STDERR_FILENO。可以查看头文件unistd.h查看相关定义。
<mark style="background: #FFB8EBA6;">文件描述符它的作用域是整个进程，当进程结束会自动释放已打开的文件描述符</mark>
范围：通常从0开始，最大值为系统限制（可通过`ulimit -n`查看）
### 文件描述符与文件的关系表
|层次|名称|说明|
|---|---|---|
|进程级|文件描述符表|每个进程独有，存放指向文件表项的指针。|
|系统级|文件表|系统全局，包含文件状态、当前偏移量等。|
|系统级|inode表|文件系统级，包含文件元数据（如权限、大小等）。|
### 常用操作函数表
| 函数           | 头文件              | 功能             | 返回值                 | 示例                                 |
| ------------ | ---------------- | -------------- | ------------------- | ---------------------------------- |
| **open()**   | `<fcntl.h>`      | 打开/创建文件        | 成功：新的fd  <br>失败：-1  | `fd = open("file.txt", O_RDONLY);` |
| **close()**  | `<unistd.h>`     | 关闭fd           | 成功：0  <br>失败：-1     | `close(fd);`                       |
| **read()**   | `<unistd.h>`     | 从fd读取数据        | 成功：读取字节数  <br>失败：-1 | `n = read(fd, buf, 1024);`         |
| **write()**  | `<unistd.h>`     | 向fd写入数据        | 成功：写入字节数  <br>失败：-1 | `n = write(fd, "hello", 5);`       |
| **dup()**    | `<unistd.h>`     | 复制fd           | 成功：新的fd             | `new_fd = dup(old_fd);`            |
| **dup2()**   | `<unistd.h>`     | 复制fd到指定值       | 成功：指定的fd            | `dup2(old_fd, 1);` # 重定向到stdout    |
| **fcntl()**  | `<fcntl.h>`      | 控制fd属性         | 取决于操作               | `fcntl(fd, F_SETFL, O_NONBLOCK);`  |
| **lseek()**  | `<unistd.h>`     | 移动文件偏移         | 成功：新偏移  <br>失败：-1   | `lseek(fd, 0, SEEK_SET);` # 移动到开头  |
| **select()** | `<sys/select.h>` | 多路复用等待fd就绪     | 成功：就绪fd数            | 监控多个fd的I/O状态                       |
| **poll()**   | `<poll.h>`       | 多路复用（改进select） | 成功：就绪fd数            | 无数量限制，效率更高                         |
### 文件描述符状态标志表
|标志类型|标志名|含义|设置方式|
|---|---|---|---|
|**访问模式**|O_RDONLY|只读|open()时指定|
||O_WRONLY|只写|open()时指定|
||O_RDWR|读写|open()时指定|
|**创建/打开**|O_CREAT|文件不存在则创建|open()时指定|
||O_EXCL|与O_CREAT同用，文件存在则失败|open()时指定|
||O_TRUNC|打开时清空文件|open()时指定|
||O_APPEND|追加模式（写操作总在末尾）|open()时指定或fcntl()设置|
|**同步/异步**|O_SYNC|写操作等待物理写入完成|open()时指定|
||O_ASYNC|启用信号驱动I/O|fcntl()设置|
|**非阻塞**|O_NONBLOCK|非阻塞I/O（立即返回）|open()时指定或fcntl()设置|
|**执行时关闭**|FD_CLOEXEC|exec()时自动关闭|open()时O_CLOEXEC或fcntl()设置|
### 查看和管理文件描述符的命令表
| 命令                       | 功能               | 示例                         |
| ------------------------ | ---------------- | -------------------------- |
| `ls -l /proc/<PID>/fd`   | 查看进程打开的所有fd      | `ls -l /proc/1234/fd`      |
| `lsof -p <PID>`          | 详细列出进程打开的文件      | `lsof -p 1234`             |
| `lsof -u <用户>`           | 查看用户打开的文件        | `lsof -u alice`            |
| `ss -tulnp`              | 查看socket相关的fd    | 显示所有网络连接                   |
| `cat /proc/<PID>/limits` | 查看进程限制（包括最大fd数）  | `cat /proc/1234/limits`    |
| `ulimit -n`              | 查看/设置shell的最大fd数 | `ulimit -n 1024` # 设置为1024 |
### 重要特性与限制表
| 特性    | 说明                                                                                  |
| ----- | ----------------------------------------------------------------------------------- |
| 继承性   | 子进程继承父进程的fd表（fork()后），但可设置FD_CLOEXEC避免继承                                            |
| 共享性   | 同一文件可被多个fd指向（同一进程或不同进程）                                                             |
| 引用计数  | 文件表项有引用计数，所有指向它的fd关闭后，文件才真正关闭                                                       |
| 最大数量  | 系统级限制：`cat /proc/sys/fs/file-max`  <br>用户级限制：`ulimit -n`  <br>进程级限制：通过getrlimit()获取 |
| 分配规则  | 总是分配当前最小的可用整数（如关闭fd 3后，下次open可能返回3）                                                 |
| 特殊值-1 | 表示无效的fd，很多系统调用对-1有特殊处理                                                              |
### 常见问题与解决方案表
|问题|原因|解决方案|
|---|---|---|
|**EMFILE (Too many open files)**|进程打开文件数超过限制|1. 检查代码是否及时close()  <br>2. 使用`ulimit -n`增大限制  <br>3. 使用`lsof`查找泄漏|
|**EBADF (Bad file descriptor)**|使用已关闭的fd|1. 确保fd有效后再使用  <br>2. 避免fd被意外关闭|
|**文件偏移共享**|dup()的fd共享偏移量|需要独立偏移时使用open()重新打开|
|**阻塞I/O卡住**|默认是阻塞模式|对于网络编程，设置O_NONBLOCK|
|**exec()后fd保持打开**|默认继承父进程fd|设置FD_CLOEXEC标志|
### 编程最佳实践表
|实践|理由|
|---|---|
|**打开后立即检查返回值**|避免使用无效的fd（-1）|
|**错误处理中关闭fd**|防止资源泄漏|
|**使用O_CLOEXEC标志**|防止exec()后意外继承fd|
|**循环中注意close()**|特别是服务器循环accept时|
|**注意fd的线程安全性**|fd在进程内共享，多线程操作需同步|
|**使用getrlimit()查询限制**|编写健壮的程序|
这个表格全面涵盖了文件描述符的核心概念、操作方法和实际应用，是理解Linux I/O机制的关键。
## 文件IO常用函数介绍
### 打开、创建文件、关闭文件
#### `open`和`openat`函数
作用：打开文件
```c
	#include<fcntl.h>
	int open(const char* path,int oflag,.../*mode_t mode*/);
	int openat(int fd,const char*path,int oflag,.../*mode_t mode */);
```
- **参数**：
  - `path`：要打开或者创建文件的名字
  - `oflag`：用于指定函数的操作行为：
    - **以下五个常量中必须指定且只能指定一个**：
      - `O_RDONLY`：文件只读打开
      - `O_WRONLY`：文件只写打开
      - `O_RDWR`：文件读、写打开
      - `O_EXEC`：只执行打开
      - `O_SEARCH`：只搜索打开（应用于目录）。本文涉及的操作系统都没有支持该常量
    
    - **以下常量是可选的（进行或运算）**：
      - `O_APPEND`：每次写时都追加到文件的尾端
      - `O_CLOEXEC`：将`FD_CLOEXEC`常量设置为文件描述符标志
      - `O_CREAT`：若此文件不存在则创建它。在使用此选项时，需要同时说明参数`mode`（指定该文件的访问权限）
      - `O_DIRECTORY`：若`path`引用的不是目录，则出错
      - `O_EXCL`：<mark style="background: #FF5582A6;">若同时指定了`O_CREAT`时，且文件已存在则出错。根据此可以测试一个文件是否存在。若不存在则创建此文件。这使得测试和创建两者成为一个原子操作</mark>
      - `O_NOCTTY`：若`path`引用的是终端设备，则不将该设备分配作为此进程的控制终端
      - `O_NOFOLLOW`：若`path`引用的是一个符号链接，则出错
      - `O_NONBLOCK`：如果`path`引用的是一个`FIFO`、一个块特殊文件或者一个字符特殊文件，则文件本次打开操作和后续的 I/O 操作设为非阻塞模式。
      - `O_SYNC`：每次 `write` 等待物理 I/O 完成，包括由 `write` 操作引起的文件属性更新所需的 I/O 
      - `O_TRUNC`：如果此文件存在，且为`O_WRONLY`或者`O_RDWR`成功打开，则将其长度截断为0
      - `O_RSYNC`：使每一个`read`操作等待，直到所有对文件同一部分挂起的写操作都完成。
      - `O_DSYNC`：每次 `write` 等待物理 I/O 完成，但不包括由 `write` 操作引起的文件属性更新所需的 I/O 

  - `mode`：文件访问权限。文件访问权限常量在 `<sys/stat.h>` 中定义，有下列九个：
    - `S_IRUSR`：用户读
    - `S_IWUSR`：用户写
    - `S_IXUSR`：用户执行
    - `S_IRGRP`：组读
    - `S_IWGRP`：组写			
    - `S_IXGRP`：组执行			
    - `S_IROTH`：其他读
    - `S_IWOTH`：其他写
    - `S_IXOTH`：其他执行

- **对于`openat`函数**，被打开的文件名由`fd`和`path`共同决定：
  - 如果`path`指定的是绝对路径，此时`fd`被忽略。`openat`等价于`open`
  - 如果`path`指定的是相对路径名，则`fd`是一个目录打开的文件描述符。被打开的文件的绝对路径由该`fd`描述符对应的目录加上`path`组合而成
  - 如果`path`是一个相对路径名，而`fd`是常量`AT_FDCWD`，则`path`相对于当前工作目录。被打开文件在当前工作目录中查找。

- **返回值**：	
  - 成功：返回文件描述符
  - 失败：返回 -1

由 `open/openat` 返回的文件描述符一定是最小的未使用的描述符数字。
#### `creat`函数
作用：创建一个新文件
```c
	#include<fcntl.h>
	int creat(const char*path,mode_t mode);
```
 - 参数：
		`path`:要创建文件的名字
		`mode`：指定该文件的访问权限文件访问权限常量在 `<sys/stat.h>` 中定义，有下列九个：
			`S_IRUSR`：用户读
			`S_IWUSR`：用户写
			`S_IXUSR`：用户执行
			`S_IRGRP`：组读
			`S_IWGRP`：组写			 
			`S_IXGRP`：组执行			
			`S_IROTH`：其他读
			`S_IWOTH`：其他写
			`S_IXOTH`：其他执行  
- 返回值：
		成功： 返回`O_WRONLY`打开的文件描述符
		失败： 返回 -1

	该函数等价于`open(path,O_WRONLY|O_CREAT|O_TRUNC,mode)`。注意：
	- 它以只写方式打开，因此若要读取该文件，则必须先关闭，然后重新以读方式打开。
	- <mark style="background: #FF5582A6;">若文件已存在则将文件截断为0。</mark>

#### `close`函数
作用：关闭文件
```c
	#include<unistd.h>
	int close(int fd);
```
- 参数：
	- `fd`：待关闭文件的文件描述符
- 返回值：
	- 成功：返回 0
	- 失败：返回 -1

注意：
- 进程关闭一个文件会释放它加在该文件上的所有记录锁。
- 当一个进程终止时，内核会自动关闭它所有的打开的文件。

### 定位、读、写文件
#### `lseek`函数
作用：设置打开文件的偏移量
```c
	#include<unistd.h>
	off_t lseek(int fd, off_t offset,int whence);
```
- 参数：
	- `fd`：打开的文件的文件描述符
	- `whence`：必须是 `SEEK_SET`、`SEEK_CUR`、`SEEK_END`三个常量之一
	- `offset`：
		- 如果 `whence`是`SEEK_SET`，则将该文件的偏移量设置为距离文件开始处`offset`个字节
		- 如果 `whence` 是 `SEEK_CUR`，则将该文件的偏移量设置为当前值加上`offset`个字节，`offset`可正，可负
		- 如果 `whence` 是 `SEEK_END`，则将该文件的偏移量设置为文件长度加上`offset`个字节，`offset`可正，可负
- 返回值：
	- 成功： 返回新的文件偏移量
	- 失败：返回 -1

每个打开的文件都有一个与其关联的“当前文件偏移量”。它通常是个非负整数，用于度量从文件开始处计算的字节数。通常读、写操作都从当前文件偏移量处开始，并且使偏移量增加所读写的字节数。注意：
- 打开一个文件时，除非指定`O_APPEND`选项，否则系统默认将该偏移量设为0
- <mark style="background: #FF5582A6;">如果文件描述符指定的是一个管道、FIFO、或者网络套接字，则无法设定当前文件偏移量，则`lseek`将返回 -1 ，并且将 `errno` 设置为 `ESPIPE`</mark>。
- 对于普通文件，其当前文件偏移量必须是非负值。但是某些设备运行负的偏移量出现。因此比较`lseek`的结果时，不能根据它小于0 就认为出错。要根据是否等于 -1 来判断是否出错。
- `lseek` 并不会引起任何 I/O 操作，`lseek`仅仅将当前文件的偏移量记录在内核中。
- 当前文件偏移量可以大于文件的当前长度。此时对该文件的下一次写操作将家常该文件，并且在文件中构成一个空洞。空洞中的内容位于文件中但是没有被写过，其字节被读取时都被读为0
文件中的空洞并不要求在磁盘上占据存储区。具体处理方式与操作系统有关

#### `read`函数
作用：读取文件内容
```c
	#include<unistd.h>
	ssize_t read(int fd,void *buf,size_t nbytes);
```
- 参数：
	- `fd`：打开的文件的文件描述符
	- `buf`：存放读取内容的缓冲区的地址（由程序员手动分配）
	- `nbytes`：期望读到的字节数
- 返回值：
	- 成功：返回读到的字节数，若已到文件尾则返回 0
	- 失败：返回 -1

读操作从文件的当前偏移量开始，在成功返回之前，文件的当前偏移量会增加实际读到的字节数。有多种情况可能导致实际读到的字节数少于期望读到的字节数：
- 读普通文件时，在读到期望字节数之前到达了文件尾端
- 当从终端设备读时，通常一次最多读取一行（终端默认是行缓冲的）
- 当从网络读时，网络中的缓存机制可能造成返回值小于期望读到的字节数
- 当从管道或者` FIFO `读时，若管道包含的字节少于所需的数量，则 `read`只返回实际可用的字节数
- 当从某些面向记录的设备（如磁带）中读取时，一次最多返回一条记录
- 当一个信号造成中断，而已读了部分数据时。
#### `write`函数
作用：向文件写数据
```c
	#include<unistd.h>
	ssize_t write(int fd,const void *buf,size_t nbytes);
```
- 参数：
	- `fd`：打开的文件的文件描述符
	- `buf`：存放待写的数据内容的缓冲区的地址（由程序员手动分配）
	- `nbytes`：期望写入文件的字节数
- 返回值：
	- 成功：返回已写的字节数
	- 失败：返回 -1

`write`的返回值通常都是与`nbytes`相同。否则表示出错。`write`出错的一个常见原因是磁盘写满，或者超过了一个给定进行的文件长度限制
对于普通文件，写操作从文件的当前偏移量处开始。如果打开文件时指定了`O_APPEND`选项，则每次写操作之前，都会将文件偏移量设置在文件的当前结尾处。在一次成功写之后，该文件偏移量增加实际写的字节数。

### 原子操作、同步、复制、修改文件描述符
#### `pread/pwrite`函数
作用：原子定位读和原子定位写
```c
	#include<unistd.h>
	ssize_t pread(int fd,void*buf,size_t nbytes,off_t offset);
	ssize_t pwrite(int fd,const void*buf,size_t nbytes,off_t offset);
```
- 参数：
	- `fd`：打开的文件描述符
	- `buf`：读出数据存放的缓冲区/ 写到文件的数据的缓冲区
	- `nbytes`：预期读出/写入文件的字节数
	- `offset`：从文件指定偏移量开始执行`read/write`
- 返回：
	- 成功：读到的字节数/已写的字节数
	- 失败： -1

调用`pread`相当于先调用`lseek`再调用`read`.但是调用`pread`时，无法中断其定位和读操作，并且不更新当前文件偏移量；调用`pwrite`相当于先调用`lseek`再调用`write`.但是调用`pwrite`时，无法中断其定位和写操作，并且不更新当前文件偏移量
#### `dup/dup2`函数
作用：复制一个现有的文件描述符
```c
	#include<unistd.h>
	int dup(int fd);
	int dup2(int fd,int fd2);
```
- 参数：
	- `fd`：被复制的文件描述符（已被打开）
	- `fd2`：指定的新的文件描述符（待生成）
- 返回值：
	- 成功： 返回新的文件描述符
	- 失败： 返回 -1

对于`dup`函数，返回的新的文件描述符一定是当前可用的文件描述符中最小的数字。对于`dup2`函数：
- 如果 `fd2`已经是被打开的文件描述符且不等于`fd`，则先将其关闭，然后再打开（<font color='red'>注意关闭再打开是一个原子操作</font>）
- 如果 `fd2`等于`fd`，则直接返回`fd2`（也等于`fd`），而不作任何操作
使用dup函数重定向标准输出到文件实例：
```c
// 1. 使用dup() - 系统分配最小可用fd
int fd = open("file.txt", O_RDONLY);
int fd_copy = dup(fd);  // 复制fd，比如返回4
// fd和fd_copy指向同一个文件表项，共享偏移量

// 2. 使用dup2() - 重定向标准输出
int log_fd = open("output.log", O_WRONLY | O_CREAT, 0644);
dup2(log_fd, STDOUT_FILENO);  // 将stdout重定向到文件
printf("这行会写入文件而不是终端\n");

// 3. 使用fcntl的F_DUPFD
int fd = open("data.bin", O_RDONLY);
int new_fd = fcntl(fd, F_DUPFD, 10);  // 返回≥10的最小可用fd
// 如果10可用则返回10，否则11、12...

// 4. 使用dup3()设置标志 (Linux特有)
int fd = open("temp.txt", O_RDWR);
int fd2 = dup3(fd, 5, O_CLOEXEC);  // 复制到fd=5，并设置CLOEXEC标志
```
#### `sync`函数
作用：同步文件到磁盘
```c
	#include<unistd.h>
	int fsync(int fd);
	int fdatasync(int fd);
	void sync(void);
```
- 参数（前两个函数）：
	- `fd`：指定的打开的文件描述符

- 返回值（前两个函数）：
	-  成功：返回 0
	- 失败： 返回 -1

区别：
- `sync`：将所有修改过的块缓冲区排入写队列，然后返回，它并不等待时机写磁盘结束
- `fsync`：只对由`fd`指定的单个文件起作用，等待写磁盘操作结束才返回
- `fdatasync`：只对由`fd`指定的单个文件起作用，等待写磁盘操作结束才返回，但是它只影响文件的数据部分（`fsync`会同步更新文件的属性）
`update` 守护进程会周期性的调用`sync`函数。命令`sync`也会调用`sync`函数

#### `fcntl`函数
作用：修改文件描述符属性
```c
	#include<fcntl.h>
	int fcntl(int fd,int cmd,.../* int arg */);
```
- 参数：
	- `fd`：已打开文件的描述符
	- `cmd`：有下列若干种：
		- `F_DUPFD`常量：复制文件描述符 `fd`。新文件描述符作为函数值返回。它是尚未打开的文件描述符中大于或等于`arg`中的最小值。新文件描述符与`fd`共享同一个文件表项，但是新描述符有自己的一套文件描述符标志，其中`FD_CLOEXEC`文件描述符标志被清除
		- `F_DUPFD_CLOEXEC`常量：复制文件描述符。新文件描述符作为函数值返回。它是尚未打开的个描述符中大于或等于`arg`中的最小值。新文件描述符与`fd`共享同一个文件表项，但是新描述符有自己的一套文件描述符标志，其中`FD_CLOEXEC`文件描述符标志被设置
		- `F_GETFD`常量：对应于`fd`的文件描述符标志作为函数值返回。当前只定义了一个文件描述符标志`FD_CLOEXEC`
		- `F_SETFD`常量：设置`fd`的文件描述符标志为`arg`
		- `F_GETFL`常量：返回`fd`的文件状态标志。文件状态标志必须首先用屏蔽字 `O_ACCMODE` 取得访问方式位，然后与`O_RDONLY`、`O_WRONLY`、`O_RDWR`、`O_EXEC`、`O_SEARCH`比较（这5个值互斥，且并不是各占1位）。剩下的还有：`O_APPEND`、`O_NONBLOCK`、`O_SYNC`、`O_DSYNC`、`O_RSYNC`、`F_ASYNC`、`O_ASYNC`
		- `F_SETFL`常量：设置`fd`的文件状态标志为 `arg`。可以更改的标志是：`O_APPEND`、`O_NONBLOCK`、`O_SYNC`、`O_DSYNC`、`O_RSYNC`、`F_ASYNC`、`O_ASYNC`
		- `F_GETOWN`常量：获取当前接收 `SIGIO`和`SIGURG`信号的进程 `ID`或者进程组 `ID`
		- `F_SETOWN`常量：设置当前接收 `SIGIO`和`SIGURG`信号的进程 `ID`或者进程组 `ID`为`arg`。若 `arg`是个正值，则设定进程 `ID`；若 `arg`是个负值，则设定进程组`ID`	
		- `F_GETLK`、`F_SETLK`、`F_SETLKW`：获取/设置文件记录锁
	- `arg`：依赖于具体的命令 

- 返回值：
	- 成功： 依赖于具体的命令
	- 失败： 返回 -1

#####  `fcntl`函数常用操作

| 修改操作             | 函数/标志                                 | 作用                        | 示例                                             |
| ---------------- | ------------------------------------- | ------------------------- | ---------------------------------------------- |
| **获取文件状态标志**     | `fcntl(fd, F_GETFL)`                  | 获取文件状态标志（如读写模式、同步标志等）     | `int flags = fcntl(fd, F_GETFL);`              |
| **设置文件状态标志**     | `fcntl(fd, F_SETFL, flags)`           | 修改文件状态标志（只能设置部分标志）        | `fcntl(fd, F_SETFL, flags \| O_NONBLOCK);`     |
| **获取文件描述符标志**    | `fcntl(fd, F_GETFD)`                  | 获取文件描述符标志（如FD_CLOEXEC）    | `int fd_flags = fcntl(fd, F_GETFD);`           |
| **设置文件描述符标志**    | `fcntl(fd, F_SETFD, flags)`           | 设置文件描述符标志                 | `fcntl(fd, F_SETFD, FD_CLOEXEC);`              |
| **获取文件锁信息**      | `fcntl(fd, F_GETLK, &lock)`           | 检查是否可以加锁，返回锁信息            | `fcntl(fd, F_GETLK, &lock);`                   |
| **设置非阻塞锁**       | `fcntl(fd, F_SETLK, &lock)`           | 尝试加锁，如果锁被占用则立即失败返回        | `fcntl(fd, F_SETLK, &lock);`                   |
| **设置阻塞锁**        | `fcntl(fd, F_SETLKW, &lock)`          | 尝试加锁，如果锁被占用则阻塞等待          | `fcntl(fd, F_SETLKW, &lock);`                  |
| **获取异步I/O所有者**   | `fcntl(fd, F_GETOWN)`                 | 获取接收SIGIO/SIGURG信号的进程/进程组 | `int owner = fcntl(fd, F_GETOWN);`             |
| **设置异步I/O所有者**   | `fcntl(fd, F_SETOWN, pid)`            | 设置接收SIGIO/SIGURG信号的进程/进程组 | `fcntl(fd, F_SETOWN, getpid());`               |
| **复制文件描述符**      | `fcntl(fd, F_DUPFD, startfd)`         | 复制fd，返回≥startfd的最小可用fd    | `int new_fd = fcntl(fd, F_DUPFD, 10);`         |
| **复制文件描述符（带标志）** | `fcntl(fd, F_DUPFD_CLOEXEC, startfd)` | 复制fd并设置FD_CLOEXEC标志       | `int new_fd = fcntl(fd, F_DUPFD_CLOEXEC, 10);` |
##### 常用修改操作代码示例
```c
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

// 1. 设置非阻塞模式
int set_nonblock(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) {
        perror("fcntl F_GETFL");
        return -1;
    }
    if (fcntl(fd, F_SETFL, flags | O_NONBLOCK) == -1) {
        perror("fcntl F_SETFL O_NONBLOCK");
        return -1;
    }
    return 0;
}

// 2. 取消非阻塞模式
int clear_nonblock(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) {
        perror("fcntl F_GETFL");
        return -1;
    }
    if (fcntl(fd, F_SETFL, flags & ~O_NONBLOCK) == -1) {
        perror("fcntl F_SETFL clear O_NONBLOCK");
        return -1;
    }
    return 0;
}

// 3. 设置追加模式
int set_append(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) return -1;
    return fcntl(fd, F_SETFL, flags | O_APPEND);
}

// 4. 设置FD_CLOEXEC标志
int set_cloexec(int fd) {
    int flags = fcntl(fd, F_GETFD, 0);
    if (flags == -1) return -1;
    return fcntl(fd, F_SETFD, flags | FD_CLOEXEC);
}

// 5. 检查文件是否为非阻塞模式
int is_nonblock(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) return -1;
    return (flags & O_NONBLOCK) ? 1 : 0;
}

// 6. 文件加锁示例
int lock_file(int fd, int lock_type) {
    struct flock lock;
    lock.l_type = lock_type;     // F_RDLCK, F_WRLCK, F_UNLCK
    lock.l_whence = SEEK_SET;    // 从文件开头计算
    lock.l_start = 0;           // 起始偏移
    lock.l_len = 0;             // 锁定整个文件
    
    // 非阻塞加锁
    if (fcntl(fd, F_SETLK, &lock) == -1) {
        if (errno == EACCES || errno == EAGAIN) {
            return 0;  // 文件已被锁定
        }
        return -1;  // 其他错误
    }
    return 1;  // 加锁成功
}

// 7. 阻塞式文件加锁
int lock_file_wait(int fd, int lock_type) {
    struct flock lock;
    lock.l_type = lock_type;
    lock.l_whence = SEEK_SET;
    lock.l_start = 0;
    lock.l_len = 0;
    
    // 阻塞加锁（等待直到获取锁）
    return fcntl(fd, F_SETLKW, &lock);
}

// 8. 获取当前锁状态
void check_lock(int fd) {
    struct flock lock;
    lock.l_type = F_WRLCK;
    lock.l_whence = SEEK_SET;
    lock.l_start = 0;
    lock.l_len = 0;
    
    if (fcntl(fd, F_GETLK, &lock) == -1) {
        perror("fcntl F_GETLK");
        return;
    }
    
    if (lock.l_type == F_UNLCK) {
        printf("文件未被锁定\n");
    } else {
        printf("文件已被进程 %d 锁定，锁类型：%s\n", 
               lock.l_pid,
               lock.l_type == F_RDLCK ? "读锁" : "写锁");
    }
}
```
