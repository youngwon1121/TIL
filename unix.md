#### Process의 생성

process를 fork()해 복사 후 exec계열의 명령어(execv, execvp, execl, execlp,,)를 통해 프로세스를 변경한다.

process는 트리를 만든다.

### ./a.out 에 ./를 붙이는 이유??

shell이 execvp또는 execlp 사용했을때 첫번째 인자로 a.out을 주게 되면 execvp와 execlp는 a.out을 $PATH에서 찾는다.

그래서 $PATH가 아닌 현재디렉토리에서 찾으라고 ./를 명시해주는것이다.



## 목차

1. [Memory Mapping](#1. Memory-Mapping)
   1. [프로세스 동기화](#프로세스-동기화)
2. PIPE
3. FIFO

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

Memory Mapping에 대한 예제코드를 보다가 이해가 되었다.

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











