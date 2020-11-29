## 3.3.4

### create

功能：给定中断结构栈指针，创建一个具有初始initial_size大小的新文件。

实现函数：

```c++
void sys_create(struct intr_frame *f)
```

> 传入参数：
>
> ```c++
> struct intr_frame *f //中断存储寄存器栈指针
> ```
>
> 操作步骤：
>
> 1. 检查地址空间大小是否足够
> 2. 调用 acquire_lock_f() 获取文件写锁
> 3. 调用 filesys_create() 创建文件
> 4. 调用 release_lock_f() 释放锁

函数流程图：

![sys_create](.\picture\sys_create.png)

### remove

功能：根据中断结构栈指针，删除已有文件。

实现函数：

```c++
void sys_remove(struct intr_frame* f)
```

> 传入参数：
>
> ```c++
> struct intr_frame *f //中断存储寄存器栈指针
> ```
>
> 操作步骤：
>
> 1. 调用 acquire_lock_f() 获取文件写锁
> 2. 调用 filesys_remove() 删除文件
> 3. 调用 release_lock_f() 释放锁

函数流程图：

![sys_remove](.\picture\sys_remove.png)

### open

功能：根据中断结构栈指针，打开已有文件。

实现函数：

```c++
void sys_open(struct intr_frame* f)
```

> 传入参数：
>
> ```c++
> struct intr_frame *f //中断存储寄存器栈指针
> ```
>
> 操作步骤：
>
> 1. 调用 acquire_lock_f() 获取文件写锁
> 2. 调用 filesys_open() 打开文件
> 3. 调用 release_lock_f() 释放锁
> 4. - 如果文件成功打开，将打开的文件放到当前线程打开的文件队列thread_file_temp中，返回文件id为当前文件池fd。
>    - 如果打开失败，文件返回fd=-1失败状态值

函数流程图：

![sys_open](.\picture\sys_open.png)

### filesize

功能：根据中断结构栈指针，获取文件大小。

实现函数：

```c++
void sys_filesize(struct intr_frame* f)
```

> 传入参数：
>
> ```c++
> struct intr_frame *f //中断存储寄存器栈指针
> ```
>
> 操作步骤：
>
> 1. 调用 acquire_lock_f() 获取文件写锁
> 2. 调用 find_file_id()根据文件fid获取文件在当前线程打开文件队列中的元素指针。 
> 3. 调用 release_lock_f() 释放锁
> 4. - 如果成功在队列中找到文件，调用file_length()获取文件长度
>    - 如果失败，文件返回-1状态值

函数流程图：

![sys_filesize](.\picture\sys_filesize.png)

### read

功能：根据中断结构栈指针，从缓冲区读取文件。

实现函数：

```c++
void sys_read(struct intr_frame* f)
```

> 传入参数：
>
> ```c++
> struct intr_frame *f //中断存储寄存器栈指针
> ```
>
> 操作步骤：
>
> 1. 通过中断存储寄存器栈指针获取文件读头和文件大小，并检查缓冲区。
>
>    - 如果读入缓冲区无效就提前通过exit_special()终止操作。
>
>    - 如果输入文件指针有效可以容纳就继续向下运行程序。
>
>      1. 如果读入状态为0，通过文件头指针，通过input_getc()函数缓冲区读入循环读入缓冲区，并赋值文件大小。
>
>      2. 如果读文件，则根据文件fid查询文件地址指针
>         - 如果指针存在，则获取文件锁，通过file_read()读取文件后释放文件锁
>         - 如果指针不存在，文件返回-1状态值。

函数流程图：

![sys_read](.\picture\sys_read.png)

### write

功能：根据中断结构栈指针，从缓冲区写入文件。

实现函数：

```c++
void sys_write(struct intr_frame* f)
```

> 传入参数：
>
> ```c++
> struct intr_frame *f //中断存储寄存器栈指针
> ```
>
> 操作步骤：
>
> 1. 通过中断存储寄存器栈指针获取文件写头和文件大小
>
> 2. 通过栈内输入的状态变量判断动作：
>
>    - 如果读入状态为1，通过文件写头指针与putbuf函数，将数据写入到缓冲区。
>
>    - 否则，则将在当前线程的文件列表中根据fid读取文件
>
>      - 如果指针存在，则获取文件锁，通过file_read()读取文件后释放文件锁。
>- 如果指针不存在，文件返回-1状态值。

函数流程图：

![sys_write](.\picture\sys_write.png)

### seek

功能：根据中断结构栈指针，调用file_seek（）操作。

实现函数：

```c++
void sys_seek(struct intr_frame* f)
```

> 传入参数：
>
> ```c++
> struct intr_frame *f //中断存储寄存器栈指针
> ```
>
> 操作步骤：
>
> 1. 根据栈内信息，利用文件id找到打开的文件指针。
> 2. 调用 acquire_lock_f() 获取文件写锁
> 3. 调用 file_seek() 指定输入的文件指针偏移量
> 4. 调用 release_lock_f() 释放锁

函数流程图：

![sys_seek](E:\学习\大三\大三上\操作系统\项目\实验\Pintos-Project2\picture\sys_seek.png)

### tell

功能：根据中断结构栈指针，查找已有文件。

实现函数：

```c++
void sys_tell(struct intr_frame* f)
```

> 传入参数：
>
> ```c++
> struct intr_frame *f //中断存储寄存器栈指针
> ```
>
> 操作步骤：
>
> 1. 根据栈内信息，利用文件id找到打开的文件指针。
> 2. 调用 acquire_lock_f() 获取文件写锁
> 3. 调用 file_tell() 获取的文件指针偏移量
> 4. 调用 release_lock_f() 释放锁

函数流程图：

![sys_tell](.\picture\sys_tell.png)

### close

功能：根据中断结构栈指针，关闭已打开文件。

实现函数：

```c++
void sys_close(struct intr_frame* f)
```

> 传入参数：
>
> ```c++
> struct intr_frame *f //中断存储寄存器栈指针
> ```
>
> 操作步骤：
>
> 1. 根据栈内信息，利用文件id找到打开的文件指针。
> 2. 调用 acquire_lock_f() 获取文件写锁
> 3. 调用 file_close() 关闭文件。
> 4. 调用 release_lock_f() 释放锁
> 5. 调用list_remove()将当前关闭文件从打开文件队列中去除
> 6. 调用free()释放关闭文件的存储空间。

函数调用图：

![sys_close](.\picture\sys_close.png)



### is_valid_pointer

功能：检查用户指针是否有效

实现函数：

```c++
bool is_valid_pointer (void* esp,uint8_t argc)
```

> 传入参数：
>
> ```c
> void* esp		// 栈顶指针
> uint8_t argc	// 页数
> ```
>
> 操作步骤：
>
> 1. 对于每个参数调用is_user_vaddr检查esp是否是有效的用户地址指针
>    - 如果没有返回false
>    - 如果有效：调用pagedir_get_page检查是否有可调用的虚拟内存空间（页）
>      - 如果失败一次，返回false。
>      - 获得全部的虚拟内存（页），返回true。

函数调用图：

![is_valid_pointer](.\picture\is_valid_pointer.png)