# The Unix Kernel
The kernel is the core of an operating system which utilizes kernel subsystems like memory management and file managment. The kernel functionality can accessed throught the kernel API (interface). 

## I/O functions

### Open

### Creat

### Close

### Lseek

### Read

### Write

Not buffered

## Files and Directories

### Stat

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
which are all included through `<sys/types.h>`. All of the datatypes are a signed integer type.

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


### Wait

### Execute

```c

```

## Environmental variables

# Information
All of the examples were developed in the programming language c and compiled with GNU c compiler.  in Ubuntu 24.04 LTS with Wsl. All of the c code was 
