# 2일차
## 5. 사용자와 사용자 그룹
### 사용자
#### 사용자의 종류
- root 사용자 : 리눅스 시스템의 모든 권한을 가진 관리자 계정
- 시스템 사용자 : 리눅스 시스템에서 만든 사용자
- 일반 사용자 : root사용자와 시스템 사용자를 제외한 모든 사용자

#### root 사용자
- 권고 사항
1. root 사용자의 비밀번호를 복잡하게 설정하고 주기적으로 변경하라
2. 시스템 외부에서 root 사용자로 로그인하지 못하게 설정하라

#### su 명령어


```bash
cat /etc/password
ottl:x:1000:1000:ottl,,,:/home/ottl:/bin/bash
```
- 의미


#### 사용자 그룹
```bash
cat /etc/group
ottl:x:1000:
```

#### 사용자 추가 및 삭제하기
```bash
adduser
deluser
```

## 6. 소유권과 권한
### 소유권
```bash
-rw-rw-r-- 1 ottl ottl 0 11월 27 10:05 1
-rw-rw-r-- 1 ottl ottl 0 11월 27 10:05 2
-rw-rw-r-- 1 ottl ottl 0 11월 27 10:05 3 # type/User/Group/Others

iam@ottl-VirtualBox:~$ echo hello >> 1
bash: 1: Permission denied
```

```bash
chown -r
```
### 파일 권한
chmod 
r: 4
w: 2
x: 1


### 디렉토리 권한
r: 폴더 내용 확인
w: 폴더 내 파일 생성
x: 폴더 열기

- .
- source

## 7. 프로세스 관리
### 프로세스란
- linux is process oriented
    - 스케줄링 엔티티
        - process : {user process}
        - thread  : {user pthread / kenal kthread}

    - "task_struct" : task(process) control bock
        - 프로세스 상태를 관리하기 위한 구조체
        - PCB(Process Controll Block)

    - instance(windows)


- active process 
- passive process



- $ps
    - -l
    - -e : entire
    - -f
- $pstree

```bash
$ ps -elf | head
F S UID          PID    PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S root           1       0  0  80   0 - 42394 -      08:57 ?        00:00:22 /sbin/init splash
1 S root           2       0  0  80   0 -     0 -      08:57 ?        00:00:00 [kthreadd]
1 I root           3       2  0  60 -20 -     0 -      08:57 ?        00:00:00 [rcu_gp]
1 I root           4       2  0  60 -20 -     0 -      08:57 ?        00:00:00 [rcu_par_gp]
1 I root           5       2  0  60 -20 -     0 -      08:57 ?        00:00:00 [slub_flushwq]
1 I root           6       2  0  60 -20 -     0 -      08:57 ?        00:00:00 [netns]
1 I root           8       2  0  60 -20 -     0 -      08:57 ?        00:00:00 [kworker/0:0H-events_highpri]
1 I root          10       2  0  60 -20 -     0 -      08:57 ?        00:00:00 [mm_percpu_wq]
1 S root          11       2  0  80   0 -     0 -      08:57 ?        00:00:00 [rcu_tasks_rude_]
```

- [] = 커널 쓰레드
- PID 1 : systemd : sbin/init splash
    - 부팅할 때 초기화를 담당
    - SysV init를 업그레이드 한 버전
- PID 2 : [kthreadd]


```bash
ottl@ottl-VirtualBox:~$ ps -l

F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
0 S  1000  115160  115159  0  80   0 -  3019 do_wai pts/0    00:00:00 bash
0 R  1000  115736  115160  0  80   0 -  3179 -      pts/0    00:00:00 ps
```
- S : 상태
- PID : process ID
- PID : Parents process ID
- PRI & NI : 우선순위

- $ sleep 100

- $ kill -l
    - $ kill -3 {PID}

### 프로세스의 생애주기
1. fork();

### 프로세스 스케줄링

### 프로세스의 상태
실행 또는 대기
중단
깨어나지 않는 수면 상태
깨어날 수 있는 수면 상태
종료


### 파일 디스크립터와 표준 스트림
파일 입출력의 종류
    1. 저수준 파일 입출력
        - 버퍼 x : for device
        - 버퍼 o : for application
    2. 고수준 파일 입출력
        - 버퍼 o : for application

#### file descriptor
```c
    int fd = open(); // read(), write(), close()
    printf("%d", fd); // 3 ...4...5 ~
```
stdin  0
stdout 1
sdderr 2
이미 3가지의 파일이 open되어 있음

#### file stream
```c
    FILE* fp = fopen(); // fread(), fwirte(), ..., fclose()
```

### Foreground와 Background 프로세스
- $ jobs
- $ fg {jos number *ex %3*}

#### fg
stdin 가능
stdout 가능
stderr 가능

#### bg
stdin 불가
stdout 가능
stderr 가능

#### $ ping
ICMP 에코 요청 및 응답

#### $ jobs
- 실행중인 유저 프로세스?

### IPC
- Inter Process Communication : 프로세스 간 통신에 사용하는 도구

#### 파이프
- 단방향

- pipe : 두 프로세스 간 데이터를 전송할 때 사용하는 IPC
- named pipe : 파일 형식의 이름을 부여할 수 있는 파이프
#### 메시지 큐
- 단방향

#### 소켓
- 양방향?

- 소켓 : 
- 인터넷 소켓 : 
- 유닉스 도메인 소켓

#### 공유 메모리
각자의 가상메모리로 동일한 물리 메모리에 접근하여 통신하는 방식?

#### 세마포어
- 카운팅 세마포어
- 바이너리 세마포어


## 8. 시그널 송수신
- 특정 프로세스에게 어떤 이벤트가 발생했음을 알려줌
```bash
$ kill -l;
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

커널 -> 프로세스
프로세스 <-> 프로세스
프로세스 자신

에게 송신할 수 있다


POSIX



# 리눅스 활용

## 9. Bash: 조건문과 테스트 연산자
### 변수 in Bash
```bash
$ V1 = test
$ echo $V1
test
```

let "표현식"
expr 표현식



### 조건문과 테스트 연산자
#### if문
if
then
elif
then
else
fi

if()와 if(( a < b ))


#### if문의 조건
- True  : 프로세스 종료 코드로 0을 리턴 받았을 때
- False : 0이 아닌 값을 받을 때

#### 문자열 테스트 연산자

#### 산술 테스트 연산자

#### 파일 테스트 연산자

#### 논리 테스트 연산자

#### case문


## 10. Bash: 반복문, 함수, 변수, 배열, 쿼팅















---

### memo
- 
