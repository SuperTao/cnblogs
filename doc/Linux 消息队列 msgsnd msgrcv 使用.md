```
// 编写一个demo用于测试消息队列函数的使用方法。
// demo的参考来源，使用man msgsnd，查看man文档，里面就有使用的方式，进行更改调试。
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

/* 这个结构体的内容是自定义的,可以是任何类型的结构体
 * 第一个成员必须是long类型。
 * 后面的成员变量自己定义
 */
struct msgbuf {
    long mtype;
    char mtext[80];
};

static void
usage(char *prog_name, char *msg)
{
    if (msg != NULL)
        fputs(msg, stderr);

    fprintf(stderr, "Usage: %s [options]\n", prog_name);
    fprintf(stderr, "Options are:\n");
    fprintf(stderr, "-s        send message using msgsnd()\n");
    fprintf(stderr, "-r        read message using msgrcv()\n");
    fprintf(stderr, "-t        message type (default is 1)\n");
    fprintf(stderr, "-k        message queue key (default is 1234)\n");
    fprintf(stderr, "-q        remove message queue by key\n");
    exit(EXIT_FAILURE);
}

static void
send_msg(int qid, int msgtype)
{
    struct msgbuf msg;
    time_t t;

    msg.mtype = msgtype;

    time(&t);
    snprintf(msg.mtext, sizeof(msg.mtext), "a message at %s\nnice to meet you! Tao",
            ctime(&t));
	/* 发送数据，数据长度不包括第一个成员变量
	 * IPC_NOWAIT:当队列满时，不阻塞
	 */
    if (msgsnd(qid, &msg, sizeof(msg.mtext),
                IPC_NOWAIT) == -1) {
        perror("msgsnd error");
        exit(EXIT_FAILURE);
    }
    printf("sent: %s\n", msg.mtext);
}

static void
get_msg(int qid, int msgtype)
{
    struct msgbuf msg;
	/*
	 * 接收数据，数据长度不包括定义结构体的第一个变量
	 * MSG_NOERROR: 发送的消息超过size，把消息截断，截断部分被丢弃，且不同之发送进程
	 * IPC_NOWAIT: 队列没有消息时，不阻塞
	 * msgtype: 0 :  接收第一个消息
	 *          >0:  接收类型等于msgtype的第一个消息
	 *          <0:  接收类型等于或者小于msgtype绝对之的第一个消息
         * 发送和接收的msgtype不一样的话，也接收不到数据。
	 */
    if (msgrcv(qid, &msg, sizeof(msg.mtext), msgtype,
               MSG_NOERROR | IPC_NOWAIT) == -1) {
        if (errno != ENOMSG) {
            perror("msgrcv");
            exit(EXIT_FAILURE);
        }
        printf("No message available for msgrcv()\n");
    } else
        printf("message received: %s\n", msg.mtext);
}

static void
remove_msg(int qid)
{
    if (msgctl(qid, IPC_RMID, 0)) {
        if (errno != ENOMSG) {
            perror("msgctl");
            exit(EXIT_FAILURE);
        }
        printf("rm message fail\n");
    } else
        printf("rm message success\n");
}

int
main(int argc, char *argv[])
{
    int qid, opt;
    int mode = 0;               /* 1 = send, 2 = receive */
    int msgtype = 1;
    int msgkey = 0x1234;
	int rm_qid;

    while ((opt = getopt(argc, argv, "srt:k:q:")) != -1) {
        switch (opt) {
        case 's':
			// 发送
            mode = 1;
            break;
        case 'r':
			// 接收
            mode = 2;
            break;
        case 't':
			// 更改类型
            msgtype = atoi(optarg);
            if (msgtype <= 0)
                usage(argv[0], "-t option must be greater than 0\n");
            break;
        case 'k':
			// 更改key值
            msgkey = atoi(optarg);
            break;
	case 'q':      // 删除
	    rm_qid = atoi(optarg);
	    mode = 3;
	    break;
        default:
            usage(argv[0], "Unrecognized option\n");
        }
    }

    if (mode == 0)
        usage(argv[0], "must use either -s or -r option\n");
	// 获取或者创建一个key，并设置权限
    qid = msgget(msgkey, IPC_CREAT | 0666);

    if (qid == -1) {
        perror("msgget");
        exit(EXIT_FAILURE);
    }
	else {
		printf("msgget success, id = %d\n", qid);
	}
	
    if (mode == 1)
        send_msg(qid, msgtype); // 接收
	else if (mode == 2)
        get_msg(qid, msgtype);  // 发送
	else if (mode == 3)
		remove_msg(rm_qid);     // 根据id进行删除

    exit(EXIT_SUCCESS);
}
```

```
ipcs进行查看

ipcrm -q id进行删除

消息队列为空
tao@T:~$ ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status

------ Semaphore Arrays --------
key        semid      owner      perms      nsems

#发送消息队列
tao@T:~$ ./a.out -s
msgget success, id = 7
sent: a message at Sun Sep 18 15:41:09 2022

nice to meet you! Tao
tao@T:~$ ipcs
# 已经创建了消息队列，分配了key，msqid，已经权限，还有使用的used-bytes，使用了80字节
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0x00001234 7          tao        666        80           1
# 共享内存是空
------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
# 信号量是空
------ Semaphore Arrays --------
key        semid      owner      perms      nsems

tao@T:~$ ./a.out -r
msgget success, id = 7
message received: a message at Sun Sep 18 15:41:09 2022

nice to meet you! Tao
tao@T:~$ ipcs
# 读取之后，used-bytes的字数为空
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0x00001234 7          tao        666        0            0

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
# 根据id值，删除队列
tao@T:~$ ./a.out -q 7
msgget success, id = 7
rm message success
tao@T:~$ ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
```
