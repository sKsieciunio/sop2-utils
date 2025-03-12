# Comprehensive Guide to Pipes and FIFOs in UNIX/Linux Systems

## Table of Contents
1. [Introduction to Pipes and FIFOs](#introduction-to-pipes-and-fifos)
2. [Anonymous Pipes](#anonymous-pipes)
3. [Named Pipes (FIFOs)](#named-pipes-fifos)
4. [Pipe Operations and Properties](#pipe-operations-and-properties)
5. [Communication Patterns](#communication-patterns)
6. [Advanced Pipe Techniques](#advanced-pipe-techniques)
7. [Error Handling](#error-handling)
8. [Performance Considerations](#performance-considerations)
9. [Pipe vs. Other IPC Mechanisms](#pipe-vs-other-ipc-mechanisms)

## Introduction to Pipes and FIFOs

Pipes and FIFOs (First-In-First-Out) are fundamental interprocess communication (IPC) mechanisms in UNIX/Linux systems. They provide a way for processes to communicate by transferring data through a byte stream.

### Key Characteristics

- **Unidirectional**: Data flows in only one direction (from writer to reader)
- **FIFO Processing**: Data is read in the same order it was written
- **Byte-oriented**: Transfer raw bytes without message boundaries
- **Limited Buffer Size**: Usually between 4KB and 64KB depending on the system
- **Blocking I/O**: Operations block by default when buffer is full/empty

### Two Types of Pipes

1. **Anonymous Pipes**: Communication between related processes (parent-child)
2. **Named Pipes (FIFOs)**: Communication between unrelated processes

## Anonymous Pipes

Anonymous pipes provide a communication channel between related processes, typically a parent and its child processes. They exist only as long as the processes that use them.

### Creating Anonymous Pipes

```c
#include <unistd.h>

int pipe(int filedes[2]);
```

- `filedes[0]`: Read end of the pipe
- `filedes[1]`: Write end of the pipe
- Returns 0 on success, -1 on error

### Basic Usage Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {
    int pipefd[2];
    pid_t pid;
    char buf[100];
    const char *message = "Hello from parent process!";
    
    // Create the pipe
    if (pipe(pipefd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }
    
    // Create child process
    pid = fork();
    
    if (pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    }
    
    if (pid == 0) {  // Child process
        // Close unused write end
        close(pipefd[1]);
        
        // Read from pipe
        read(pipefd[0], buf, sizeof(buf));
        printf("Child received: %s\n", buf);
        
        // Close read end
        close(pipefd[0]);
        exit(EXIT_SUCCESS);
    } else {  // Parent process
        // Close unused read end
        close(pipefd[0]);
        
        // Write to pipe
        write(pipefd[1], message, strlen(message) + 1);
        
        // Close write end
        close(pipefd[1]);
        
        // Wait for child to complete
        wait(NULL);
        exit(EXIT_SUCCESS);
    }
    
    return 0;
}
```

### Pipe File Descriptors

- File descriptors for pipes behave like regular file descriptors
- Can be used with standard I/O functions (read, write, close)
- Can be redirected with dup/dup2
- Can be passed to select/poll for I/O multiplexing

### Pipe2 Function (Linux-specific)

```c
#include <fcntl.h>
#include <unistd.h>

int pipe2(int filedes[2], int flags);
```

Allows setting flags during pipe creation:
- `O_NONBLOCK`: Set non-blocking I/O
- `O_CLOEXEC`: Close-on-exec flag

```c
// Create non-blocking pipe
if (pipe2(pipefd, O_NONBLOCK) == -1) {
    perror("pipe2");
    exit(EXIT_FAILURE);
}
```

## Named Pipes (FIFOs)

Named pipes, or FIFOs, are pipes with a presence in the filesystem. They allow communication between unrelated processes and persist until explicitly deleted from the filesystem.

### Creating Named Pipes

From the command line:
```bash
mkfifo /path/to/myfifo
```

In C code:
```c
#include <sys/types.h>
#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode);
```

- `pathname`: Path where the FIFO will be created
- `mode`: Permission bits (similar to file permissions)
- Returns 0 on success, -1 on error

### FIFO Usage Example

Writer process:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>

int main() {
    int fd;
    const char *fifo_path = "/tmp/myfifo";
    const char *message = "Hello through named pipe!";
    
    // Create the FIFO if it doesn't exist
    mkfifo(fifo_path, 0666);
    
    // Open FIFO for writing
    fd = open(fifo_path, O_WRONLY);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }
    
    // Write to the FIFO
    write(fd, message, strlen(message) + 1);
    
    // Close FIFO
    close(fd);
    
    return 0;
}
```

Reader process:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>

int main() {
    int fd;
    const char *fifo_path = "/tmp/myfifo";
    char buffer[100];
    
    // Open FIFO for reading
    fd = open(fifo_path, O_RDONLY);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }
    
    // Read from the FIFO
    read(fd, buffer, sizeof(buffer));
    printf("Received: %s\n", buffer);
    
    // Close FIFO
    close(fd);
    
    return 0;
}
```

### FIFO Characteristics

- Exists as a special file in the filesystem
- Persistence: Remains until explicitly unlinked
- Permissions: Can be set and changed like regular files
- Blocking behavior: Opening for read blocks until writer opens, and vice versa (by default)

### Non-blocking FIFO Operations

```c
// Open FIFO in non-blocking mode
fd = open(fifo_path, O_RDONLY | O_NONBLOCK);

// Check for EAGAIN error
if (read(fd, buffer, sizeof(buffer)) == -1) {
    if (errno == EAGAIN) {
        printf("No data available\n");
    } else {
        perror("read");
    }
}
```

## Pipe Operations and Properties

### Reading and Writing

```c
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
```

- Atomic up to PIPE_BUF bytes (typically 4KB)
- Partial reads/writes can occur depending on buffer space

### Handling EOF

- When all write ends are closed, read returns 0 (EOF)
- When all read ends are closed, write raises SIGPIPE signal

```c
#include <signal.h>

// Handle SIGPIPE
void sigpipe_handler(int sig) {
    printf("SIGPIPE received\n");
}

// In main()
signal(SIGPIPE, sigpipe_handler);
```

Or ignore SIGPIPE and check for EPIPE error:

```c
// Ignore SIGPIPE
signal(SIGPIPE, SIG_IGN);

// Check for EPIPE
if (write(fd, buf, count) == -1) {
    if (errno == EPIPE) {
        printf("No readers available\n");
    }
}
```

### Pipe Capacity

The pipe buffer size can be queried and (on some systems) modified:

```c
#include <unistd.h>
#include <fcntl.h>

// Get pipe buffer size (Linux-specific)
int size = fcntl(pipefd[0], F_GETPIPE_SZ);
printf("Pipe buffer size: %d bytes\n", size);

// Set pipe buffer size (Linux-specific)
fcntl(pipefd[0], F_SETPIPE_SZ, 65536);  // Set to 64KB
```

## Communication Patterns

### One-way Communication

The basic pattern shown earlier: parent writes, child reads, or process A writes to a FIFO, process B reads from it.

### Two-way Communication (Bidirectional)

Requires two pipes:

```c
int pipe_parent_to_child[2];
int pipe_child_to_parent[2];

pipe(pipe_parent_to_child);
pipe(pipe_child_to_parent);

if (fork() == 0) {  // Child
    // Close unused ends
    close(pipe_parent_to_child[1]);
    close(pipe_child_to_parent[0]);
    
    // Use pipe_parent_to_child[0] for reading from parent
    // Use pipe_child_to_parent[1] for writing to parent
    
} else {  // Parent
    // Close unused ends
    close(pipe_parent_to_child[0]);
    close(pipe_child_to_parent[1]);
    
    // Use pipe_parent_to_child[1] for writing to child
    // Use pipe_child_to_parent[0] for reading from child
}
```

### Multiple Readers/Writers

Multiple processes can read from or write to the same pipe:

```c
if (fork() == 0) {  // Child 1
    // Write to pipe
} else if (fork() == 0) {  // Child 2
    // Also write to pipe
} else {  // Parent
    // Read from pipe (will receive data from both children)
}
```

### Redirection with dup2

Redirecting standard I/O to pipes:

```c
// Replace stdout with pipe write end
dup2(pipefd[1], STDOUT_FILENO);
close(pipefd[1]);  // Original descriptor no longer needed

// Now printf will write to the pipe
printf("This goes to the pipe\n");
```

### Creating Pipeline Between Commands

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    int pipefd[2];
    pid_t pid1, pid2;
    
    // Create pipe
    pipe(pipefd);
    
    // First child (ls)
    pid1 = fork();
    if (pid1 == 0) {
        // Redirect stdout to pipe
        dup2(pipefd[1], STDOUT_FILENO);
        
        // Close unused pipe ends
        close(pipefd[0]);
        close(pipefd[1]);
        
        // Execute ls
        execlp("ls", "ls", "-l", NULL);
        perror("execlp ls");
        exit(EXIT_FAILURE);
    }
    
    // Second child (grep)
    pid2 = fork();
    if (pid2 == 0) {
        // Redirect stdin from pipe
        dup2(pipefd[0], STDIN_FILENO);
        
        // Close unused pipe ends
        close(pipefd[0]);
        close(pipefd[1]);
        
        // Execute grep
        execlp("grep", "grep", "^d", NULL);
        perror("execlp grep");
        exit(EXIT_FAILURE);
    }
    
    // Parent closes pipe ends
    close(pipefd[0]);
    close(pipefd[1]);
    
    // Wait for children
    waitpid(pid1, NULL, 0);
    waitpid(pid2, NULL, 0);
    
    return 0;
}
```

## Advanced Pipe Techniques

### Multiplexing I/O with select/poll

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/select.h>

int main() {
    int pipe1[2], pipe2[2];
    fd_set readfds;
    char buffer[100];
    
    // Create pipes
    pipe(pipe1);
    pipe(pipe2);
    
    // Child process 1
    if (fork() == 0) {
        close(pipe1[0]);
        write(pipe1[1], "Message from child 1", 20);
        exit(0);
    }
    
    // Child process 2
    if (fork() == 0) {
        close(pipe2[0]);
        sleep(2);  // Delay to demonstrate select
        write(pipe2[1], "Message from child 2", 20);
        exit(0);
    }
    
    // Parent process
    close(pipe1[1]);
    close(pipe2[1]);
    
    // Prepare for select
    int maxfd = (pipe1[0] > pipe2[0]) ? pipe1[0] : pipe2[0];
    
    while (1) {
        FD_ZERO(&readfds);
        FD_SET(pipe1[0], &readfds);
        FD_SET(pipe2[0], &readfds);
        
        // Wait for data from either pipe
        if (select(maxfd + 1, &readfds, NULL, NULL, NULL) == -1) {
            perror("select");
            exit(EXIT_FAILURE);
        }
        
        // Check which pipe has data
        if (FD_ISSET(pipe1[0], &readfds)) {
            int nbytes = read(pipe1[0], buffer, sizeof(buffer));
            if (nbytes <= 0) {
                close(pipe1[0]);
                FD_CLR(pipe1[0], &readfds);
            } else {
                buffer[nbytes] = '\0';
                printf("From pipe 1: %s\n", buffer);
            }
        }
        
        if (FD_ISSET(pipe2[0], &readfds)) {
            int nbytes = read(pipe2[0], buffer, sizeof(buffer));
            if (nbytes <= 0) {
                close(pipe2[0]);
                FD_CLR(pipe2[0], &readfds);
            } else {
                buffer[nbytes] = '\0';
                printf("From pipe 2: %s\n", buffer);
            }
        }
        
        // Exit if both pipes are closed
        if (!FD_ISSET(pipe1[0], &readfds) && !FD_ISSET(pipe2[0], &readfds)) {
            break;
        }
    }
    
    return 0;
}
```

### Using poll()

```c
#include <poll.h>

struct pollfd fds[2];
fds[0].fd = pipe1[0];
fds[0].events = POLLIN;
fds[1].fd = pipe2[0];
fds[1].events = POLLIN;

while (1) {
    int ret = poll(fds, 2, -1);  // -1 for infinite timeout
    
    if (ret == -1) {
        perror("poll");
        exit(EXIT_FAILURE);
    }
    
    if (fds[0].revents & POLLIN) {
        // Read from pipe1
    }
    
    if (fds[1].revents & POLLIN) {
        // Read from pipe2
    }
}
```

### Passing File Descriptors

On Unix systems, you can pass file descriptors between processes using a Unix domain socket. This is more advanced than basic pipes but useful for complex IPC:

```c
#include <sys/socket.h>
#include <sys/un.h>

// Send fd to another process
ssize_t send_fd(int socket, int fd_to_send) {
    struct msghdr msg = {0};
    struct cmsghdr *cmsg;
    char iobuf[1];
    struct iovec io = {
        .iov_base = iobuf,
        .iov_len = sizeof(iobuf)
    };
    union {
        char buf[CMSG_SPACE(sizeof(int))];
        struct cmsghdr align;
    } u;
    
    msg.msg_iov = &io;
    msg.msg_iovlen = 1;
    msg.msg_control = u.buf;
    msg.msg_controllen = sizeof(u.buf);
    
    cmsg = CMSG_FIRSTHDR(&msg);
    cmsg->cmsg_level = SOL_SOCKET;
    cmsg->cmsg_type = SCM_RIGHTS;
    cmsg->cmsg_len = CMSG_LEN(sizeof(int));
    
    *((int *) CMSG_DATA(cmsg)) = fd_to_send;
    
    return sendmsg(socket, &msg, 0);
}

// Receive fd from another process
int recv_fd(int socket) {
    struct msghdr msg = {0};
    struct cmsghdr *cmsg;
    char iobuf[1];
    struct iovec io = {
        .iov_base = iobuf,
        .iov_len = sizeof(iobuf)
    };
    union {
        char buf[CMSG_SPACE(sizeof(int))];
        struct cmsghdr align;
    } u;
    int fd;
    
    msg.msg_iov = &io;
    msg.msg_iovlen = 1;
    msg.msg_control = u.buf;
    msg.msg_controllen = sizeof(u.buf);
    
    if (recvmsg(socket, &msg, 0) < 0)
        return -1;
        
    cmsg = CMSG_FIRSTHDR(&msg);
    if (cmsg == NULL)
        return -1;
        
    if (cmsg->cmsg_level != SOL_SOCKET || cmsg->cmsg_type != SCM_RIGHTS)
        return -1;
        
    fd = *((int *) CMSG_DATA(cmsg));
    return fd;
}
```

## Error Handling

Common errors when working with pipes:

### SIGPIPE Signal

Sent to a process writing to a pipe without readers:

```c
// Ignore SIGPIPE
signal(SIGPIPE, SIG_IGN);

// Or handle it
signal(SIGPIPE, sigpipe_handler);
```

### EPIPE Error

Similar to SIGPIPE, but checked via errno:

```c
if (write(pipefd[1], buffer, nbytes) == -1) {
    if (errno == EPIPE) {
        printf("No readers available\n");
    } else {
        perror("write");
    }
}
```

### EAGAIN Error

In non-blocking mode, indicates operation would block:

```c
if (read(pipefd[0], buffer, sizeof(buffer)) == -1) {
    if (errno == EAGAIN) {
        printf("No data available\n");
    } else {
        perror("read");
    }
}
```

### Comprehensive Error Handling Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <string.h>
#include <signal.h>

void sigpipe_handler(int signum) {
    printf("Caught SIGPIPE\n");
}

int main() {
    int pipefd[2];
    char buffer[1024];
    ssize_t nread, nwritten;
    
    // Set up SIGPIPE handler
    struct sigaction sa;
    sa.sa_handler = sigpipe_handler;
    sa.sa_flags = 0;
    sigemptyset(&sa.sa_mask);
    sigaction(SIGPIPE, &sa, NULL);
    
    // Create pipe
    if (pipe(pipefd) == -1) {
        fprintf(stderr, "pipe failed: %s\n", strerror(errno));
        exit(EXIT_FAILURE);
    }
    
    // Set non-blocking I/O
    int flags = fcntl(pipefd[0], F_GETFL);
    fcntl(pipefd[0], F_SETFL, flags | O_NONBLOCK);
    
    // Try reading from empty pipe (will return EAGAIN)
    nread = read(pipefd[0], buffer, sizeof(buffer));
    if (nread == -1) {
        if (errno == EAGAIN) {
            printf("No data available (expected)\n");
        } else {
            fprintf(stderr, "read failed: %s\n", strerror(errno));
            exit(EXIT_FAILURE);
        }
    }
    
    // Write to pipe
    const char *message = "Test message";
    nwritten = write(pipefd[1], message, strlen(message) + 1);
    if (nwritten == -1) {
        fprintf(stderr, "write failed: %s\n", strerror(errno));
        exit(EXIT_FAILURE);
    }
    
    // Now read should succeed
    nread = read(pipefd[0], buffer, sizeof(buffer));
    if (nread == -1) {
        fprintf(stderr, "read failed: %s\n", strerror(errno));
        exit(EXIT_FAILURE);
    } else {
        printf("Read %zd bytes: %s\n", nread, buffer);
    }
    
    // Close read end
    close(pipefd[0]);
    
    // Try writing to pipe without readers (will trigger SIGPIPE/EPIPE)
    nwritten = write(pipefd[1], message, strlen(message) + 1);
    if (nwritten == -1) {
        if (errno == EPIPE) {
            printf("Write to pipe with no readers (EPIPE detected)\n");
        } else {
            fprintf(stderr, "write failed: %s\n", strerror(errno));
        }
    }
    
    // Close write end
    close(pipefd[1]);
    
    return 0;
}
```

## Performance Considerations

### Buffer Size

Larger pipe buffers can improve performance for large data transfers:

```c
// On Linux systems
fcntl(pipefd[0], F_SETPIPE_SZ, 1024 * 1024);  // Set to 1MB
```

### Write Size

Writes smaller than PIPE_BUF (typically 4KB) are atomic:

```c
// Guaranteed to be atomic
write(pipefd[1], buffer, 4096);

// May be split if pipe doesn't have enough space
write(pipefd[1], large_buffer, 16384);
```

### Non-blocking I/O

For high-performance applications, consider non-blocking I/O with select/poll/epoll:

```c
// Set non-blocking mode
fcntl(pipefd[0], F_SETFL, O_NONBLOCK);
fcntl(pipefd[1], F_SETFL, O_NONBLOCK);
```

### Memory Copying

Pipes involve kernel buffer copy operations. For very large data, consider:
- Memory-mapped files
- Shared memory
- Unix domain sockets with `SCM_RIGHTS` to pass file descriptors

## Pipe vs. Other IPC Mechanisms

| Mechanism | Pros | Cons |
|-----------|------|------|
| **Pipes** | Simple, well-understood, file descriptor based | Unidirectional, related processes only (anonymous pipes) |
| **FIFOs** | Named in filesystem, unrelated processes can communicate | Unidirectional, limited buffer size |
| **Message Queues** | Structured messages, bidirectional | More complex API |
| **Shared Memory** | Fastest for large data | Requires synchronization |
| **Unix Domain Sockets** | Bidirectional, file descriptor passing | More complex than pipes |
