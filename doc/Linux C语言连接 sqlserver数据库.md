记录一下Linux下使用C语言连接sqlserver的方法。

连接前需要安装freetds。

参考：

http://www.cnblogs.com/helloworldtoyou/p/6910075.html

http://blog.csdn.net/neighbor1000/article/details/8824084

http://blog.csdn.net/pinkbean/article/details/64439759

这个是从参考链接中的提取代码。分析一下，写上注释。

```
#include<stdio.h>  
#include<string.h>  
#include<stdlib.h>  
#include<unistd.h>  
#include<sybfront.h>  
#include<sybdb.h>  

int main(void)  
{    
    char szUsername[32]= "Tony";  
    char szPassword[32]= "tony";  
    char szDBName[32]= "gpio";  
    char szServer[32]= "192.168.1.221:1433";        //服务器ip地址和端口号  
  
    //初始化db-library  
    dbinit();  
    //连接数据库  
    LOGINREC *loginrec=dblogin();  
	// 设置登录的用户名
    DBSETLUSER(loginrec,szUsername);  
	// 设置登录密码
    DBSETLPWD(loginrec,szPassword);  
	// 连接sqlserver服务器地址和端口号,这里才是连接
    DBPROCESS *dbprocess=dbopen(loginrec,szServer);  
    if(dbprocess==FAIL){  
        printf("Connect MSSQLSERVER fail\n");  
        return 0;  
    }else{  
        printf("Connect MSSQLSERVER success\n");  
    }  
	// 连接数据库
    if(dbuse(dbprocess,szDBName)==FAIL){  
        printf("Open data basename fail\n");  
    }else{  
        printf("Open data basename success\n");  
    }  
    // 查询数据库中表中的内容
    dbcmd(dbprocess,"select * from real_date_log");  //查询数据表中的内容，更具实际情况更改数据表名称  
	// 执行命令
    if(dbsqlexec(dbprocess)==FAIL){  
        printf("Query table error\n");  
    }  
    else   
        printf("Query table success\n");  
  
    DBINT result_code;  
    char szID[1024];  
    char szBeginTime[1024];  
    char szDescription[1024];  
	// 查看命令执行的结果。
    while((result_code=dbresults(dbprocess))!=NO_MORE_RESULTS)  
    {  
        if(result_code==SUCCEED){  
			// 查询第1列
            dbbind(dbprocess,1,CHARBIND,(DBINT)0,(BYTE*)szID);  
			// 查询第2列
            dbbind(dbprocess,2,CHARBIND,(DBCHAR)0,(BYTE*)szBeginTime);  
			// 查询第3列
            dbbind(dbprocess,3,CHARBIND,(DBCHAR)0,(BYTE*)szDescription);  
			// 跳到下一行
            while(dbnextrow(dbprocess)!=NO_MORE_ROWS){  
				// 打印本行的数据
                printf("ID=%s\n",szID);  
                printf("szAid=%s\n",szBeginTime);  
                printf("szBeginTime=%s\n",szDescription);  
            }  
        }  
    }  
  
    //关闭数据库连接  
    dbclose(dbprocess);  
    return 0;  
}
```
编译

`gcc -o testsybase testsybase.c -L /usr/local/freetds7.0/lib/ -lsybdb -I /usr/local/freetds7.0/include/`  

执行
```
export LD_LIBRARY_PATH=/usr/local/freetds7.0/lib/  
./testsybase 
```

Tony Liu

2017-5-26, Shenzhen