# The Unix Kernel
The kernel is the core of an operating system which utilizes kernel subsystems like memory management and file managment. The kernel functionality can accessed throught the kernel API (interface). 

## I/O functions

### Errno
Errno is a global variable that is set by functions supporting errno on failiure. Errno stores valuable debug information as a error code. Errno should not be accessed directly but actually has helper functions to use it.

Include: `<stdio.h>`

```c
void perror(const char* s);   // Print a system error message
```

#### Example:
```c
#include <stdio.h>
#include <unistd.h>

int main() {
    // Frok initializes errno on failiure
    if (fork() == -1) {
        perror("fork");
        return 1;
    }
}
```

### Open

Opens a file and returns a file descriptor.

Include: `<fcntl.h>`

### Creat

### Close

### Lseek

### Read

### Write

Writes bytes up to the `count` given from the buffer to the file given by the file descriptor `fd`. If the `count` is greater than SSIZE_MAX, the result can be unpredictable.

Write is not buffered which means that it does not depend on flushing of a buffer. This can make it less performant compared to for example `printf()`.

Include: `<unistd.h>`

#### Returns:
On success, returns the number of bytes written.
On failure, returns -1. `errno` is set to indicate the error.

```c
ssize_t write(int fd, const void buf[.count], size_t count);
```

#### Example:
```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("print 1\n");
    write(STDOUT_FILENO, "print 2\n", 8);
}
```

#### Output:
```
print 2
print 1
```

In the example write gets displayed first because it is not buffered while printf is. Printf's buffer is flushed at the end of main if not flushed by the user.

## Files and Directories

### File descriptors
A file descriptor is 

The files are stored in a processes process table as a file table where each file has a unique spot with flags, current file offset and v-node pointer. Files are often defined with a integer called a file descriptor which is used as a position in the file table. When a file is opened a file table entry is opened and v-node is created.

If two processes have the same file open and one of the processes close it, the file will still remain open for the other processes. When using open in two different processes each file descriptor will point to a different file table but both of the file tables will point to the same v-node. On close in one of the processes the file table will be closed but the v-node will still be open because it was shared with another processes file table. When fork is called the parent and child use the same file table because the child gets a copy of the parent.

#### Process table entry (file table):
| fd | flags | offset | v-node |
|----|-------|--------|--------|
| 0  |       |        |        |
| 1  |       |        |        |
| 2  |       |        |        |


#### V-node:
| v-node info |
|-------------|
| i-node info |
|  file size  |

can be opened and closed with `open()` and `close()`.

### Dup
Duplicates a file descriptor.

`dup(int oldfd)` can duplicate a file descriptor.

#### Return:
On success, returns the new file descriptor.
On failure, returns -1 and `errno` is set to indicate the error.

Include: `<unistd.h>`

```c
int dup(int oldfd);
int dup2(int oldfd, int newfd);
```

#### Example:
```c
#include <unistd.h>
#include <fcntl.h>

int main() {
    int fd;
    
    if ((fd = open("file", O_WRONLY | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR)) < 0) {
        perror("open");
        return 1;
    }

    if (dup2(fd, STDOUT_FILENO) < 0) {
        perror("dup2");
        return 1;
    }
}
```

#### Output:
```
Todo: write something here
```

### Stat
Returns the information about a file. The information is stored in a struct `stat` which stores different data about the file.

```c
struct stat {
    mode_t    st_mode;	    // The permissions of the file
    ino_t     st_ino;	    // The inode of the file
    dev_t     st_dev;	    // The device the file is stored on.
    uid_t     st_uid;	    // The User ID of the file.
    gid_t     st_gid;	    // The Group ID of the file.

    timespec  st_atime;	    // The last time the file was accessed.
    timespec  st_ctime;	    // The last time the file's permission was changed.
    timespec  st_mtime;	    // The last time the file was modified.

    nlink_t   st_nlink;	    // The number of links to the file.
    off_t     st_size;      // The size of the file
    blksize_t st_blksize;   // The block size for file system I/O
    blkcnt_t  st_blocks;    // The number of blocks allocated
};
```

Include: `<sys/stat.h>`

#### Return:
On success, returns 0.
On Failure, returns -1. `errno` is set appropriately

```c
int stat(const char *restrict path, struct stat *restrict statbuf);

int fstat(int fd, struct stat *statbuf);

int lstat(const char *restrict path, struct stat *restrict statbuf);
```

### File types

## Programs
Programs in unix can be executed with a variety of exec functions. Each function has a different usecase but all of them are used to execute programs.

## Processes

### Process ID
A kernel process is a action that corresponds to a process id(pid). These process id's are used in order to identify and use the process.

The common datatypes used for storing a id's are:
* `pid_t` - A process id
* `uid_t` - A user id
* `gid_t` - A group id
which are all included through `<sys/types.h>`. All of the datatypes are of signed integer type.

Every UNIX system has a implemenataion with a set of kernel processes that are used for operating the system functionalities. Below is a set of functions used for retrieving these process id's.

Include: `<unistd.h>`

```c
pid_t getpid(void);    // Returns the process ID of calling process

pid_t getppid(void);   // Returns the parent process ID of the calling process

uid_t getuid(void);    // Returns the real user ID of the calling process

uid_t geteuid(void);   // Returns the effective user ID of the calling process

gid_t getgid(void);    // Returns the real group ID of the calling process

gid_t getegid(void);   // Returns the effective group ID of the calling process
```

### Fork
Fork creates a new process by duplicating the process its called from. The new duplicated process is called the child process and the the calling process is called the parent process.

What makes this possible is that fork is called once but returns twice. It returns both from the parent and the child which makes it possible to identify what process is being called.

Each process has a seperate memory space which eliminates allocations performed by a process interfering with other processes. When fork is called both processes have a identical memory space.

#### Return:
On success, returns the pid of the child process in the parent and returns 0 in the child.
On failure, the parent return -1 and no child process is created. `errno` is set to indicate the error.

Include: `<unistd.h>`

```c
pid_t fork(void);
```

#### Example:
```c
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();

    switch (pid) {
        case -1: // Failure
            perror("fork");
            return 1;
        case 0: // Child process
            puts("Child executing");
            break;
        default: // Parent process
            puts("Parent executing");
    }
}
```

#### Output:
```
Child executing
Parent executing
```

### Exit

Causes normal process termination and returns a given status to the parent. All open stdio streams are flushed and closed.

The status can be given by these macros:
```c
EXIT_SUCCESS
EXIT_FAILURE
```

Include: `<stdlib.h>`

#### Return:
Nothing

```c
void exit(int status);
```

#### Example:
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    exit(EXIT_SUCCESS);
    printf("This should not print\n");
}
```

#### Output:
```

```

In the example the main function is exited before the `printf()` so nothing is displayed.

### Wait

Waits for status change in a child of the calling process and obtains information about the changed child. A state change can be: the child terminated, the hild was stopped by a signal or the child was resumed by a signal. If a child is terminated, wait releases the resources associated with the child. If wait is not called, the terminated child becomes a "zombie".

If the `statloc` argument is not null, the status of the terminated process is stored in the passed in pointer. To ignore this, set it as null.

Include: `<sys/wait.h>`

```c
pid_t wait(int *statloc);      // Returns on child process terminated.
pid_t waitpid(pid_t pid, int *statloc, int options);
```

There are some macros given in `<sys/wait.h>` that can interpret the termination status.

```c
WIFEXITED(status)        // True if the child terminated normally (by calling exit or returning from main()).
WIFSIGNALED(status)      // True if the child process was terminated by a signal.
WIFSTOPPED(status)       // True if the child was stopped by delivery a signal.
WIFCONTINUED(status)     // True if the child process was resumed by delivery of SIGCONT.
```

### Execute

Include: `<unistd.h>`

```c
int execl(const char *pathname, const char *arg0, ... /* (char *)0 */ );

int execv(const char *pathname, char *const argv[]);

int execle(const char *pathname, const char *arg0, ... /* (char *)0, char *const envp[] */ );

int execve(const char *pathname, char *const argv[], char *const envp[]);

int execlp(const char *filename, const char *arg0, ... /* (char *)0 */ );

int execvp(const char *filename, char *const argv[]);
```

### Pipe

Creates a connection where data can be sent between to file descriptors. The pipe ends can be closed with `close()`.

Returns:
On success, returns 0
On failure, returns -1. `errno` is set to indicate the error.

Include: `<unistd.h>`

Usage: create a child process and set up the pipe tp the parent so that it can send data to the child.
* call pipe()
* do a fork to get child.
* close childs write
* close parent read

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

#define READ_END 0
#define WRITE_END 1

// Calls perror and closes the given pipe
void perror_pipe(char* s, int* pipe) {
    perror(s);
    close(pipe[0]);
    close(pipe[1]);
}

int main() {
    int parent_to_child[2];

    pid_t pid;
    int number = 2;

    // Create a pipe
    if (pipe(parent_to_child) != 0) {
        perror("pipe");
        return 1;
    }

    pid = fork();
    switch (pid) {
        case -1:
            perror_pipe("free", parent_to_child);
            return 1;
        case 0:
            // Read from parent
            int temp_num;
            if (read(parent_to_child[READ_END], &temp_num, sizeof(int)) == -1) {
                perror_pipe("read", parent_to_child);
                return 1;
            }

            printf("%d\n", temp_num);
            break;
        default:
            // Write to child
            if (write(parent_to_child[WRITE_END], &number, sizeof(int)) == -1) {
                perror_pipe("write", parent_to_child);
                return 1;
            }

            // Wait for the child to read the data
            int status;
            if (wait(&status) == -1) {
                perror_pipe("wait", parent_to_child);
                return 1;
            }
    }

    // Close the read end of the pipe
    close(parent_to_child[READ_END]);

    // Close the write end of the pipe
    close(parent_to_child[WRITE_END]);
}
  
```

## Environmental variables

## Useful commands

* `grep word_to_search *.c` - Searches all files in the current directory for the word `word_to_search` in all files ending with `.c`.
* `tr "[a-z]" "[A-Z]" < srcfile > destfile` -
* ``who | grep jacob | wc -l` - Counts how many "lines" with jacob gotten by who.
  
# Information
All of the examples were developed in the programming language c and compiled with GNU c compiler.  in Ubuntu 24.04 LTS with Wsl. All of the c code was 
