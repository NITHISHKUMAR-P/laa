# laa
## Exp-2 Linux-Process API - fork(), wait(), exec()
### Printing Process ID and parent Process ID
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(void)
{	
	pid_t process_id;
	pid_t p_process_id;	
	process_id = getpid();
	p_process_id = getppid();

	printf("The process id: %d\n",process_id);
	printf("The process id of parent function: %d\n",p_process_id);
	return 0; 
 }
 ```
### Create new process using Linux API system calls fork() and exit():
```c
#include <stdlib.h>
#include <sys/wait.h>
#include<stdio.h>
#include<unistd.h>
#include <sys/types.h>

int main()
{
   int pid; 
   pid=fork(); 
   if(pid == 0) {
      printf("Iam child my pid is %d\n",getpid()); 
      printf("My parent pid is:%d\n",getppid()); 
      exit(0);
   } 
   else{ 
      printf("I am parent, my pid is %d\n",getpid()); 
      sleep(100); 
      exit(0);
   } 
}
```
### C Program to execute Linux system commands using Linux API system calls exec() family:
```c
#include <stdlib.h>
#include <sys/wait.h>
#include<stdio.h>
#include<unistd.h>
#include <sys/types.h>
int main()
{       int status;
        printf("Running ps with execlp\n");
        execl("ps", "ps", "ax", NULL);
        wait(&status);
        if (WIFEXITED(status))
                printf("child exited with status of %d\n", WEXITSTATUS(status));
        else
                puts("child did not exit successfully\n");
        printf("Done.\n");
        printf("Running ps with execlp. Now with path specified\n");
        execl("/bin/ps", "ps", "ax", NULL);
        wait(&status);
        if (WIFEXITED(status))
                printf("child exited with status of %d\n", WEXITSTATUS(status));
        else
                puts("child did not exit successfully\n");
        printf("Done.\n");
        exit(0);
}
```
## Exp-3 IPC Pipes
### C Program that illustrate communication between two process using unnamed pipes using Linux API system calls:
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/wait.h>

void server(int, int);
void client(int, int);

int main() {
    int p1[2], p2[2], pid, *waits;
    pipe(p1);
    pipe(p2);
    pid = fork();
    if (pid == 0) {
        close(p1[1]);
        close(p2[0]);
        server(p1[0], p2[1]);
        return 0;
    }
    close(p1[0]);
    close(p2[1]);
    client(p1[1], p2[0]);
    wait(waits);
    return 0;
}

void server(int rfd, int wfd) {
    int i, j, n;
    char fname[2000];
    char buff[2000];
    n = read(rfd, fname, 2000);
    fname[n] = '\0';
    int fd = open(fname, O_RDONLY);
    sleep(10);
    if (fd < 0)
        write(wfd, "can't open", 9);
    else
        n = read(fd, buff, 2000);
    write(wfd, buff, n);
}

void client(int wfd, int rfd) {
    int i, j, n;
    char fname[2000];
    char buff[2000];
    printf("ENTER THE FILE NAME :");
    scanf("%s", fname);
    printf("CLIENT SENDING THE REQUEST .... PLEASE WAIT\n");
    sleep(10);
    write(wfd, fname, 2000);
    n = read(rfd, buff, 2000);
    buff[n] = '\0';
    printf("THE RESULTS OF CLIENTS ARE ...... \n");
    write(1, buff, n);
}
```
### C Program that illustrate communication between two process using named pipes using Linux API system calls:
```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>

int main() {
    int res = mkfifo("/tmp/my_fifo", 0777);
    if (res == 0)
        printf("FIFO created\n");
    exit(EXIT_SUCCESS);
}
```
