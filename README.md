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
## Ex.No: 4 Linux-IPC-Message-Queues
### C program that receives a message from message queue and display them (writer.c):
```c
// C Program for Message Queue (Writer Process) 
#include <stdio.h> 
#include <stdlib.h> // for exit()
#include <sys/ipc.h> 
#include <sys/msg.h> 

struct mesg_buffer { 
	long mesg_type; 
	char mesg_text[100]; 
} message; 

int main() 
{ 
	key_t key; 
	int msgid;
	key = ftok("progfile", 65); 

	msgid = msgget(key, 0666 | IPC_CREAT); 
	message.mesg_type = 1; 

	printf("Write Data : "); 
	fgets(message.mesg_text, sizeof(message.mesg_text), stdin); 
	if (msgsnd(msgid, &message, sizeof(message), 0) == -1) {
		perror("msgsnd");
		exit(EXIT_FAILURE);
	}
	printf("Data sent is : %s \n", message.mesg_text); 
	return 0; 
}
```
#### reader.c:
```c
// C Program for Message Queue (Reader Process)
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/msg.h>

struct mesg_buffer {
	long mesg_type;
	char mesg_text[100];
} message;
int main()
{
	key_t key;
	int msgid;
	key = ftok("progfile", 65);
	msgid = msgget(key, 0666 | IPC_CREAT);
	msgrcv(msgid, &message, sizeof(message), 1, 0);
	printf("Data Received is : %s \n",message.mesg_text);

	msgctl(msgid, IPC_RMID, NULL);
	return 0;
}
```
## Ex.No: 5 Linux-IPC-Semaphores
### Write a C program that implements a producer-consumer system with two processes using Semaphores:
```c
#include <stdio.h>	 
#include <stdlib.h>      
#include <unistd.h>	 
#include <time.h>	 
#include <sys/types.h>   
#include <sys/ipc.h>     
#include <sys/sem.h>	 

#define NUM_LOOPS	20	

#if defined(__GNU_LIBRARY__) && !defined(_SEM_SEMUN_UNDEFINED)

#else

union semun {
        int val;                    
        struct semid_ds *buf;       
        unsigned short int *array;  
        struct seminfo *__buf;      
};
#endif

int main(int argc, char* argv[]) {
    int sem_set_id;	      
    union semun sem_val;      
    int child_pid;	      
    int i;		      
    struct sembuf sem_op;     
    int rc;		      
    struct timespec delay;    

    sem_set_id = semget(IPC_PRIVATE, 1, 0600);
    if (sem_set_id == -1) {
        perror("main: semget");
        exit(1);
    }

    printf("semaphore set created, semaphore set id '%d'.\n", sem_set_id);

    sem_val.val = 0;
    rc = semctl(sem_set_id, 0, SETVAL, sem_val);

    child_pid = fork();
    switch (child_pid) {
        case -1:
            perror("fork");
            exit(1);
        case 0:
            for (i = 0; i < NUM_LOOPS; i++) {
                sem_op.sem_num = 0;
                sem_op.sem_op = -1;
                sem_op.sem_flg = 0;
                semop(sem_set_id, &sem_op, 1);
                printf("consumer: '%d'\n", i);
                fflush(stdout);
            }
            break;
        default:
            for (i = 0; i < NUM_LOOPS; i++) {
                printf("producer: '%d'\n", i);
                fflush(stdout);
                sem_op.sem_num = 0;
                sem_op.sem_op = 1;
                sem_op.sem_flg = 0;
                semop(sem_set_id, &sem_op, 1);

                if (rand() > 3*(RAND_MAX/4)) {
                    delay.tv_sec = 0;
                    delay.tv_nsec = 10;
                    nanosleep(&delay, NULL);
                }

                if (NUM_LOOPS >= 10) {
                    semctl(sem_set_id, 0, IPC_RMID, NULL);
                }
            }
            break;
    }
    return 0;
}
```
## Ex06-Linux IPC-Shared-memory
### Write a C program that illustrates two processes communicating using shared memory:
```c
// Write a C program that illustrates two processes communicating using shared memory.

#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>

int main()
{
	key_t key = ftok("shmfile", 65);

	int shmid = shmget(key, 1024, 0666 | IPC_CREAT);
        printf("Shared memory id = %d \n",shmid);
	char* str = (char*)shmat(shmid, (void*)0, 0);
	
    	printf("Write Data : ");
	fgets(str, 1024, stdin);

	printf("Data written in memory: %s\n", str);
	shmdt(str);
	return 0;
}
```
## Ex07-Linux-File-IO-Systems-locking
### To Write a C program that illustrates files copying:
```c
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h> // Include stdio.h for perror()

int main() {
    char block[1024];
    int in, out;
    int nread;

    in = open("filecopy.c", O_RDONLY);

    out = open("file.out", O_WRONLY | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR);

    exit(EXIT_SUCCESS);
}
```
### To Write a C program that illustrates files locking:
```c
#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/file.h>

int main(int argc, char* argv[]) {
    if (argc != 2) {
        printf("Usage: %s <filename>\n", argv[0]);
        return 1;
    }

    char* file = argv[1];
    int fd;
    struct flock lock;

    printf("Opening %s\n", file);
  
    fd = open(file, O_RDWR);
    if (fd == -1) {
        perror("Error opening file");
        return 1;
    }

    lock.l_type = F_RDLCK; 
    lock.l_whence = SEEK_SET;
    lock.l_start = 0;
    lock.l_len = 0;
    if (fcntl(fd, F_SETLK, &lock) == -1) {
        perror("Error acquiring shared lock");
    } else {
        printf("Acquiring shared lock using fcntl\n");
    }
    getchar();

    lock.l_type = F_WRLCK; 
    if (fcntl(fd, F_SETLK, &lock) == -1) {
        perror("Error acquiring exclusive lock");
    } else {
        printf("Acquiring exclusive lock using fcntl\n");
    }
    getchar();

    lock.l_type = F_UNLCK;
    if (fcntl(fd, F_SETLK, &lock) == -1) {
        perror("Error releasing lock");
    } else {
        printf("Unlocking\n");
    }
    getchar();

    close(fd);
    return 0;
}
```
## Exp-8 Advanced Batch Scripting:
### Create a batch script named "BackupScript.bat" that creates a backup of files with the ".docx" extension from the "Documents" folder to a new folder named "DocBackup" on the desktop:
```bat
@echo off
set "source_folder=%USERPROFILE%\Documents"
set "destination_folder=%USERPROFILE%\Desktop\DocBackup"

echo Creating backup folder...
mkdir "%destination_folder%" 2>nul

echo Copying .docx files from Documents folder to DocBackup folder...
xcopy "%source_folder%\*.docx" "%destination_folder%\" /s /i /y >nul

echo Backup completed.
pause
```
