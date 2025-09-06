# The Unix Kernel
The kernel is the core of an operating system which utilizes kernel subsystems like memory management and file managment. The kernel functionality can accessed throught the kernel API (interface). 

## Programs
Programs in unix can be executed with a variety of exec functions. Each function has a different usecase but all of them are used to execute programs.

## Processes
A kernel process is a action that corresponds to a process id(pid). These process id's are used in order to get process status and being able to use the process.

The common datatype used for storing a process id is `pid_t` which is included through `<sys/types.h>`. It is a signed integer type.

### Fork
Fork creates a new process by duplicating the process its called from. The new duplicated process is called the child process and the the calling process is called the parent process.

Each process has a seperate memory space which eliminates allocations performed by a process interfering with other processes. When fork is called both processes have a identical memory space.

#### Return:
On success, returns the pid of the child process in the parent and returns 0 in the child.
On failure, the parent return -1 and no child process is created. `errno` is set to indicate the error.

Include: `<unistd.h`

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

  switch (pid);
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

### Execute

```c

```

## Environmental variables

# Information
All of the examples were developed in the programming language c and compiled with GNU c compiler.  in Ubuntu 24.04 LTS with Wsl. All of the c code was 
