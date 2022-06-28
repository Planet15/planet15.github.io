---
layout: post
title:  "systemd TimeoutSec라는 옵션에 대한 검증 service 예제 파일 구현하기"
date:   2022-06-28
last_modified_at: 2022-06-28
categories: [linux]
tags: [linux]
---

systemd 에 사용되는 서비스의 특정 값에 대해서 확인이 필요하여
testservice 라는 서비스를 구현하여 값을 검증 하고자 한다.

현재의 구현 의도는 systemd에서 제공하는 TimeoutSec의기능을 검증
하기 위해서 이다.

```c
testservice.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <time.h>
#include <string.h>
#include <signal.h>

void (*old_fun)( int);

//TimeoutSec 기능검증용
//SIGTERM을받으면 무한loop 으로 구현
void sigint_handler( int signo)
{
        if (signo == SIGTERM)
        {
                printf("SIGANL SIGTERM recived!\n");
                while(1)
                {

                }
        }
}

int main(int argc, char* argv[])
{
        FILE *fp,*fp1= NULL;
        pid_t process_id = 0;
        pid_t sid = 0;
        old_fun = signal(SIGTERM, sigint_handler);

        // Create child process
        process_id = fork();
        // Indication of fork() failure
        if (process_id < 0)
        {
                printf("fork failed!\n");
                // Return failure in exit status
                exit(1);
        }
        // PARENT PROCESS. Need to kill it.
        if (process_id > 0)
        {
		//프로세스 관리를 위해 pid 파일을구성
                fp1 = fopen ("/tmp/testservice.pid", "w+");
                fprintf(fp1, "%d", process_id);
                fflush(fp1);
                fclose(fp1);
                exit(0);
        }

        //unmask the file mode
        umask(0);
        //set new session
        sid = setsid();
        if(sid < 0)
        {
                // Return failure
                exit(1);
        }
        // Change the current working directory to root.
        chdir("/");
        // Close stdin. stdout and stderr
        close(STDIN_FILENO);
        close(STDOUT_FILENO);
        close(STDERR_FILENO);
        // Open a log file in write mode.
        fp = fopen ("/tmp/Log.txt", "w+");
        while (1)
        {
                char datetime[16];
                memset(datetime, 0x00, sizeof(datetime));
                //Dont block context switches, let the process sleep for some time
                sleep(1);
                fprintf(fp, "[ %d ] Logging info...\n", getDateTime(datetime));
                fflush(fp);
                // Implement and call some function that does core work for this daemon.
        }
        fclose(fp);
        return (0);
}

//현재 time을 가지고 오기 위해 구현
int getDateTime(char* datetime)
{
        struct tm *t;
        time_t tnow;
        tnow = time(NULL);
        t= (struct tm*) localtime(&tnow);
        return (int)tnow;
}
```

이후 간단히 gcc를 활용하여 컴파일을 한다.

```sh
# gcc -o testservice testservice.c
# cp -rf testservice /usr/sbin/testservice

```

systemd 에 아래와 같이 등록을 한다.

```sh
/etc/systemd/system/testservice.service
[Unit]
Description=example systemd service unit file.

[Service]
PIDFile=/tmp/testservice.pid
ExecStart=/usr/sbin/testservice
ExecStop=/usr/bin/kill -15 $MAINPID
TimeoutSec=300

[Install]
WantedBy=multi-user.target
```

systemd에 testservice의 구동 후, 상태를 확인한다.
```sh
# systemctl start testservice.service
# systemctl status testservice.service
● testservice.service - example systemd service unit file.
   Loaded: loaded (/etc/systemd/system/testservice.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2022-06-24 16:19:33 KST; 4s ago
  Process: 4236 ExecStop=/usr/bin/kill -15 $MAINPID (code=exited, status=0/SUCCESS)
 Main PID: 4273 (testservice)
   CGroup: /system.slice/testservice.service
           └─4273 /usr/sbin/testservice

Jun 24 16:19:33 localhost.localdomain systemd[1]: Started example systemd service unit file..
```

이후 systemctl stop testservice.service 입력 후 5분 뒤 종료를 확인 한다.
아래와 같이 Active 항목 시간에  4min 59s ago 시간을 확인 할수있다.

```sh
[root@localhost ~]# systemctl stop testservice.service

5분후
[root@localhost ~]# systemctl status testservice.service
● testservice.service - example systemd service unit file.
   Loaded: loaded (/etc/systemd/system/testservice.service; enabled; vendor preset: disabled)
   Active: deactivating (stop-sigterm) since Fri 2022-06-24 16:20:20 KST; 4min 59s ago 
  Process: 4305 ExecStop=/usr/bin/kill -15 $MAINPID (code=exited, status=0/SUCCESS)
 Main PID: 4273 (testservice)
   CGroup: /system.slice/testservice.service
           └─4273 /usr/sbin/testservice

Jun 24 16:19:33 localhost.localdomain systemd[1]: Started example systemd service unit file..
Jun 24 16:20:20 localhost.localdomain systemd[1]: Stopping example systemd service unit file....
```
