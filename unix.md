#### Process의 생성

process를 fork()해 복사 후 exec계열의 명령어(execv, execvp, execl, execlp,,)를 통해 프로세스를 변경한다.

process는 트리를 만든다.

### ./a.out 에 ./를 붙이는 이유??

shell이 execvp또는 execlp 사용했을때 첫번째 인자로 a.out을 주게 되면 execvp와 execlp는 a.out을 $PATH에서 찾는다.

그래서 $PATH가 아닌 현재디렉토리에서 찾으라고 ./를 명시해주는것이다.



### signal

컴퓨터구조론 배울때 interrupt handler에 대해 다룬적이 있다.

CPU는 매 명령어를 실행한 후 인터럽트가 있는지 체크하고 인터럽트가 발생했으면 Interrupt Handler를 실행시킨다는 내용이었다.

terminal에서 무심코 누르는 컨트롤z나 컨트롤c가 바로 signal(interrupt)였던것이다.

```c
int main() {
	struct sigaction act;
	act.sa_handler = catchint;
	sigaction(SIGUSR1, &act, NULL);
  
  act.sa_handler= SIGIGN;
  sigaction(SIGINT, &act, NULL);
  
  act.sa_handler= SIGDFL;
  sigaction(SIGINT, &act, NULL);
}
```





## 목차

1. [Memory Mapping](#1. Memory-Mapping)
   1. [프로세스 동기화](#프로세스-동기화)
2. PIPE
3. FIFO
4. Message Queue
5. Semaphore
6. SharedMemory
7. Lock

---

##  1. Memory Mapping

일반적으로 우리가 파일을 읽고쓰는 open(), read(), write()등의 커널 함수들을 호출하면 DISK에 대한 access가 계속해서 이뤄진다.

Memory Mapping을 사용하면 그 오버헤드가 줄어든다.

그리고 여러 프로세스가 Memory Mapping을 사용함으로써 IPC로써의 기능도 가능하다.

Memory Mapping시 파일의 모든 부분을 다 mapping할 필요는 없으니 offset과 length를 지정해 Mapping한다.

이때 offset사이즈는 페이지의 크기와 동일해야한다고 한다.

이게 뭔소릴까? 

우리 컴퓨터는 물리메모리보다 큰 용량의 프로세스를 실행하기 위해 **가상 메모리**를 사용한다.

가상메모리 <-> 물리 메모리의 변환을 위해 페이징 시스템을 사용하는데 가상메모리의 주소를 페이지 테이블을 통해 물리메모리의 프레임으로 변환하는 과정이있다.. 

이때 가상메모리의 페이지 크기와 물리메모리의 프레임 크기는 동일하다.



## 1.1. 프로세스 동기화

운영체제 배울때 프로세스 동기화에 대해 직관적인 이해가 잘 되지 않았었다.

Memory Mapping에 대한 예제코드를 보다가 이해가 되어 첨부한다.

### 1.1.1. 동기화 되지 않는 경우

```c
//reader.c
//메모리 매핑 후, 숫자 10개를 읽어 저장
int main() {
    int fd, i;
    int *addr;
    fd = open("temp", O_RDWR|O_CREAT);
    addr = mmap(NULL, sizeof (int) * 10, PROT_WRITE, MAP_SHARED, fd, 0);

    ftruncate(fd, sizeof (int) * 10);
    for (i=0; i<10; i++){
        scanf("%d", addr+i);
    }
    exit(0);
}
```

```c
//writer.c
//위 프로그램과 같은 파일에 매핑 후, 위에서 읽은 숫자를 출력
int main(void){
    int fd, i;
    int *addr;
    fd = open("temp", O_RDONLY); // 파일을 읽기 가능하게 open
    addr = mmap(NULL, sizeof (int)*10, PROT_READ, MAP_SHARED, fd, 0);// memory mapping
            sleep(5);
    for (i=0; i<5; i++){
        printf("%d\n", *(addr+i));
    }
    sleep(5);
    for (i=5; i<10; i++){
        printf("%d\n", *(addr+i));
    }
    exit(0);
}
```

**reader.c** 는 숫자를 읽어 저장하고 

**writer.c** 는 reader.c가 읽어서 저장한 숫자를 출력하는 것이다. 즉 reader와 writer간의 데이터가 공유되는 IPC의 예제이다.

이때 writer는 reader가 언제 데이터를 읽어서 저장할지 모르기 때문에 읽는 타이밍을 sleep(5)로 고정해서 정하고있다.

5초내에 reader가 scanf를 통해 입력받지 않으면 writer는 제대로된 데이터를 전달받지 못할 것이다.

이 때 프로세스 사이의 통신이 제대로 동기화 되지 못한것이다.



### 1.1.2. 동기화 되는 경우

동기화하기 위해서 signal을 사용한 예제코드를 살펴보자.

프로세스간 signal을 보내기 위해서는 pid를 알아야하므로 pid를 주고받는 memory map도 하나 더 만들었다.

writer 프로세스는 pause()를 통해 waiting상태로 변했다가  reader가 사용자로 부터 입력을 받았을때만 signal을 받아 wakeup()되는 걸 확인 할 수있다.

위의 경우와 다르게 동기화 되는걸 볼 수 있었다.

```c
//reader.c
int main(void){
    char *addr;
    int fd1, fd2, i, n, len=0;
    pid_t *pid;
    fd1=open("data", O_RDWR|O_CREAT, 0644);// 파일을 읽기, 쓰기 가능하게 open
    fd2=open("temp", O_RDWR|O_CREAT, 0644);// 파일을 읽기, 쓰기 가능하게 open
    pid= mmap(NULL, sizeof (pid_t), PROT_READ, MAP_SHARED, fd2, 0);// memory mapping
    addr=mmap(NULL, 512, PROT_WRITE, MAP_SHARED, fd1, 0);// memory mapping
    ftruncate(fd1, 512);
    ftruncate(fd2, sizeof (int));

    while (*pid==0);
    printf("writer id = %ld\n", *pid);
    for (i=0; i<3; i++){
        // read로 입력받은 내용을
        // file에 저장
        n = read(0, addr+len, 512-len);
        if (len>=512)
            break;
        // writer에게 signal 보내기
        kill(*pid, SIGUSR1);
    }
    exit(0);
}
```



```c
//writer.c
void catchsig(int signo);
int main(void){
    char *addr;
    int fd1, fd2, i, n, len=0;
    pid_t *pid;
    static struct sigaction act;

    // signal handler 설정
    act.sa_handler = catchsig;
    sigaction(SIGUSR1, &act, NULL);

    fd1=open("data", O_RDWR|O_CREAT, 0644);// 파일을 읽기, 쓰기 가능하게 open
    fd2=open("temp", O_RDWR|O_CREAT, 0644);// 파일을 읽기, 쓰기 가능하게 open
    pid= mmap(NULL, sizeof (pid_t), PROT_WRITE, MAP_SHARED, fd2, 0);// memory mapping
    addr=mmap(NULL, 512, PROT_READ, MAP_SHARED, fd1, 0);// memory mapping
    //
    *pid = getpid();

    for (i=0; i<3; i++){
        pause();
        // 읽은 내용을 write로 화면 출력
        n = write(1, addr+len, 512-len);
        n += len;
        write(1, "-------\n", 8);
        if (len>512)
            break;
    }
    exit(0);
}
void catchsig(int signo){
}
```



메모리맵핑은 실제 disk의 파일의 일정부분을  프로세스의 영역에 맵핑해오는 것이다. 

(그래서 IPC의 목적으로 메모리맵핑을 사용한다면 ftruncate나 truncate 함수로 사용할 크기를 미리 확보해주어야한다.)

그러므로 해당 영역에 쓰고지우는게 실제 파일에 저장된다. 



## 2. PIPE

프로세스 간, 단방향 통신이 가능하다..

```c
//파이프의 생성
int main() {
	int fd[2];
	pipe(fd);
}
```

- fd[0] : 읽기용
- fd[1] : 쓰기용



##### 특징

- blocking read, write
- 한 파이프에 대해 reader가 여러명일수있다.

부모 Process가 한파이프에  1, 2, 3, 4, 5를 write하고

자식 프로세스 5개에서 각자 한번씩 read 하면

자식들중 누군가가 처음 read할때 1, 그다음이 2, 3, 4, 5를 가져가는 식이다.

- Writer가 없는경우, read를 기다리는 Reader들에게는 0을 리턴,  Reader가 없는경우, writer들은 SIGPIPE signal을 받는다.





## 3. FIFO

```#include <stdio.h>```

```c
//FIFO생성
int main() {
	int p[2];
	mkfifo(p, 0666);
	open();
}
```

## 4. Message Queue

message queue는 pipe와는 다르게 양방향이다.

- PIPE, FIFO는 read(), write()함수를 사용함. 그러나 메세지큐는 msgsnd(), msgrcv()사용.

그러나 pipe와 마찬가지로 쓰인 데이터는 한번만 읽히고 사라진다.

```
//Message Queue생성
int main() {
	key = ftok()
	qid = msgget(key, IPC_CREAT|0666);
	
	//msg send
	
	msgsnd(qid,buf, size);
	msgrcv(qid, buf, size, 0);
	
	//msg rcv
}
```





## 5. Semaphore

세마포어 코드를 작성하다보면 몇개의 세마포어가 필요한건지 정해야한다.

이때 필요한 세마포어의 갯수는 <b>대기가 필요한 상황의 갯수</b> 이다.

```c
int main() {
	semid = semget(key, 2, 0600|IPC_CREAT|IPC_EXCL);

  if(semid == -1) {
    semget(key,2,0 );
  } else {
    arg.buf =buf;
    semctl(semid,  0, SETALL, arg);
  }
}
```



## 6. SharedMemory

공유메모리는 여러프로세스에서 접근할수 있도록 메모리에 공간을 할당하는것이다.

메모리에 일정한 size만큼의 공간을 할당한다.

여러프로세스간의 정보를 전달하기에는 가장 적합한 방식이다.

```c
int main () {
		key2=ftok("key", 23);
    // 공유메모리 생성 및 초기화
    shmid = shmget(key2, sizeof (int) * 10, 0600|IPC_CREAT);
    buf = (int*)shmat(shmid, 0, 0);
}
```



#### Memory Mapping과 SharedMemory 의 차이점

Memory Mapping은 file의 일부분을 <b>프로세스 내 </b>공간에 할당하는것이다.

반면 SharedMemory는 프로세스와 독립적인 메모리영역에 여러 프로세스가 접근할수있는 메모리를 할당하는것이다.

그림으로 보자.



![1](/Users/user/Workspace/TIL/IMG/1.png)

Shared Memory



![2](/Users/user/Workspace/TIL/IMG/2.png)

Memory Mappings



## 7. LOCK

file descripter의 일정부분을 락을 설정할수 있다.

하지만 lock을 건다고 다른프로세스에서 접근할수 없는것은 아니다.

만약 다른 프로세스에서 lock없이 해당 자원에 접근해 수정한다면 수정이 된다.

그러므로 동시접근을 막기위해서는 꼭 접근하는 모든부분에서 lock을 통한 접근제어가 이뤄져야한다.

lock type

- F_WRLCK

- F_RDLCK

- F_UNLCK

F_SETLK

F_SETLKW : 리턴값이 -1이면 deadlock detecting



```c
int main() {
  struct flock lock;
  lock.l_type = F_RDLCK;
  lock.l_whence = SEEK_CUR;
  lock.l_start = 0;
  lock.l_len = sizeof(int);
  fcntl(fd, F_SETLKW, &lock);
}
```
