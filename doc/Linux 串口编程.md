今天对应用层串口编程进行了验证。程序来源于以下参考链接，自己进行了一些注释和更改，记录于此。

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　Tony Liu, 2016-6-17, Shenzhen

### 参考链接

https://www.ibm.com/developerworks/cn/linux/l-serials/

http://digilander.libero.it/robang/rubrica/serial.htm

http://blog.csdn.net/querdaizhi/article/details/7436722

### 程序

```
#include     <stdio.h>      /*标准输入输出定义*/
#include     <stdlib.h>     /*标准函数库定义*/
#include     <unistd.h>     /*Unix 标准函数定义*/
#include     <sys/types.h>  
#include     <sys/stat.h>   
#include     <fcntl.h>      /*文件控制定义*/
#include     <termios.h>    /*PPSIX 终端控制定义*/
#include     <errno.h>      /*错误号定义*/
#include	 <string.h>

#define 	FALSE  	-1
#define 	TRUE   	0

void set_speed(int fd, int speed)
{
    int i; 
    int status; 
    struct termios Opt;

    tcgetattr(fd, &Opt); 		 //读取当前串口的配置, 后面会定其中的一些配置更改再写回去。
    tcflush(fd, TCIOFLUSH);      //清空输入输出队列
    cfsetispeed(&Opt, speed);    //设置输入速度 
    cfsetospeed(&Opt, speed);    //设置输出速度
    status = tcsetattr(fd, TCSANOW, &Opt);  //写给底层寄存器
    if (status != 0) {        
      perror("tcsetattr fd");  
      return;     
    }    
    tcflush(fd,TCIOFLUSH);   
}

/**
 * 设置串口数据位，停止位和效验位
 * fd        类型  int  打开的串口文件句柄
 * databits  类型  int 数据位   取值 为 7 或者8
 * stopbits  类型  int 停止位   取值为 1 或者2
 * parity    类型  int  效验类型 取值为N,E,O,,S
 */
int set_Parity(int fd,int databits,int stopbits,int parity)
{ 
    struct termios options; 
    if  (tcgetattr( fd,&options)  !=  0) { 
        perror("SetupSerial 1");     
        return(FALSE);  
    }
 
    options.c_cflag |= CLOCAL | CREAD;
    options.c_cflag &= ~CSIZE; 
    options.c_cflag &= ~CRTSCTS;            //无硬件流控  
 
    options.c_oflag = 0;                    //输出模式  
    options.c_lflag = 0;                    //不激活终端模式  

    switch (databits) /*设置数据位数*/
    {   
        case 7:     
            options.c_cflag |= CS7; 
            break;
        case 8:     
            options.c_cflag |= CS8;
            break;   
        default:    
            fprintf(stderr,"Unsupported data size\n"); return (FALSE);  
    }
    switch (parity) 
    {   
        case 'n':
        case 'N':    
            options.c_cflag &= ~PARENB;   /* Clear parity enable */
            options.c_iflag &= ~INPCK;     /* Enable parity checking */ 
            break;  
        case 'o': 
        case 'O':    
            options.c_cflag |= (PARODD | PARENB); /* 设置为奇效验*/  
            options.c_iflag |= INPCK;             /* Disnable parity checking */ 
            break;  
        case 'e':  
        case 'E':   
            options.c_cflag |= PARENB;     /* Enable parity */    
            options.c_cflag &= ~PARODD;   /* 转换为偶效验*/     
            options.c_iflag |= INPCK;       /* Disnable parity checking */
            break;
        case 'S': 
        case 's':  /*as no parity*/  	//空闲(space)校验和不校验的设置一样
            options.c_cflag &= ~PARENB;
            options.c_iflag &= ~INPCK;     /* Enable parity checking */ 
            break;  
        default:   
            fprintf(stderr,"Unsupported parity\n");    
            return (FALSE);  
    }  
    /* 设置停止位, 这里为什么没有设置1.5的停止位 */  
    switch (stopbits)
    {   
        case 1:    
            options.c_cflag &= ~CSTOPB;  
            break;  
        case 2:    
            options.c_cflag |= CSTOPB;  
            break;
        default:    
            fprintf(stderr,"Unsupported stop bits\n");  
            return (FALSE); 
    } 
	//总感觉这里的判断不是很合理，只是判断‘n’，“N”难道就不判断了？
    /* Set input parity option */ 
//    if (parity != 'n')   
//        options.c_iflag |= INPCK; 
    tcflush(fd,TCIFLUSH);
    options.c_cc[VTIME] = 150; /* 设置超时15 seconds*/   
    options.c_cc[VMIN] = 0; /* Update the options and do it NOW */
    if (tcsetattr(fd,TCSANOW,&options) != 0)   
    { 
        perror("SetupSerial 3");   
        return (FALSE);  
    } 
    return (TRUE);  
}

int OpenDev(char *Dev)
{
    int fd = open(Dev, O_RDWR | O_NOCTTY);         		  //阻塞
//    int fd = open(Dev, O_RDWR | O_NOCTTY | O_NDELAY);   // 非阻塞
    if (-1 == fd)   
    {           
        perror("Can't Open Serial Port");
        return -1;      
    }   
    else    
        return fd;
}

int main(int argc, char **argv){
    int fd;
    int nread;
    char buff[512];

    char *dev  = "/dev/ttymxc2";      //串口，根据自己板子的节点名称进行更改

    fd = OpenDev(dev);
    set_speed(fd, B9600);			//根据系统定义的波特率的宏进行设置 

    if (set_Parity(fd,8,1,'N') == FALSE)  {
        printf("Set Parity Error\n");
        exit (0);
    }

    //循环读取数据
    while (1) {   
//        nread = write(fd, "hello", 6);
//        printf("write count: %d\n", nread);
		//每次就只能读到一字节, 所以输出的Len为1，buff也只是单个字节
        while ((nread = read(fd, buff, 512))>0)
        { 
            printf("Len %d\n",nread);
            buff[nread+1] = '\0';
            printf( "%s\n", buff); 
        }
    }
    //close(fd);
    // exit (0);
}
```