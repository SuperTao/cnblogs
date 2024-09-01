使用c#读写txt文件，记录Log。

###### 参考链接：

http://www.runoob.com/csharp/csharp-file-io.html

http://www.runoob.com/csharp/csharp-text-files.html

http://www.runoob.com/csharp/csharp-binary-files.html

###### 举例

```
    StreamWriter writeLog;
    writeLog = new StreamWriter("log.txt");
    writeLog.WriteLine("hello world");
    writeLog.Flush();   //清空缓存，否者文件的内容不会实时更新
    writeLog.close();

    StreamReader readLog 
    readLog = new StreamReader("log.txt"))
    string line;
    line = sr.ReadLine();
    Console.WriteLine(line);
    readLog.close();
```

streamwriter对于文件属性的设置方法不是很多,混合FileStream使用。

```
     StreamWriter writeLog;
     StreamWriter writeTempLog;
     StreamReader readTempLog;
     FileStream Log;

     Log = new FileStream("log.txt", FileMode.Create, FileAccess.ReadWrite,FileShare.ReadWrite);
     writeLog = new StreamWriter(Log);

     readTempLog = new StreamReader(LogTemp);
     string logData;
     logData = readTempLog.ReadLine();
     while ((logData = readTempLog.ReadLine()) != null)
     {
         writeLog.WriteLine(logData);
         writeLog.Flush();
     }
     readTempLog.Close();
     writeLog.Close();
     Log.Close();
```

###### Author:

Tony Liu

2016-7-4, Shenzhen