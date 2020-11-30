#### 2.1.1 Data structure

- 在`<thread/thread.h>`增添新的结构类型 `child`和`thread_file`。

  - struct child 结构体

    ```C
    // thread/thread.h
    struct child
      {
        tid_t tid;	// ID number of the thread 
        bool isrun;	// Whether the child's thread is running successfully
        struct list_elem child_elem;	// an element of list childs
        struct semaphore sema;          // semaphore to control wait signal
        int store_exit;                 // exit state of the child thread 
      };
    ```

  - struct thread_file结构体

    ```C
    // thread/thread.h
    struct thread_file
      {
        int fd;						// file description
        struct file* file;			 // opened file pointer
        struct list_elem file_elem;   // element of the file_owned
      };
    ```

-  在`<thread/thread.h>`的`pintos`原有结构体`thread`中增添几个字段

```C
struct list childs;                 // The list of struct child elements
struct child * thread_child;        // Store a child of this thread
int st_exit;                        // Exit state of this thread
struct semaphore sema;	// Control the child process for not waiting for child
bool success;			// Whether the child's thread executes successfully
struct thread* parent;	// Parent thread of this thread
```

- 在`<thread/thread.h>`中`struct thread`中增添几个字段。

```C
struct list files;			  // List of opened files
int file_fd;				 // File's descriptor
struct file * file_owned;	  // current opened file's pointer
```


#### 