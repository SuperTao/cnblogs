运行qt时，点击QMessageBox的确定按钮是出现错误QWSLock::up(): Invalid argument, QWSLock::down(): Invalid argument，这个是qt的bug。在网上找到patch更改之后从新编译，更改之后，问题解决。

参考链接：

http://www.mail-archive.com/community@lists.openmoko.org/msg65512.html

https://git.congatec.com/yocto/poky/commit/d1b523e7dd55b11d1b02904a83a0464b95616c07

patch的内容如下：

```
diff --git a/src/gui/embedded/qwslock.cpp b/src/gui/embedded/qwslock.cpp
index 9914a24..1055785 100644
--- a/src/gui/embedded/qwslock.cpp
+++ b/src/gui/embedded/qwslock.cpp
@@ -83,9 +83,12 @@ QWSLock::QWSLock(int id) : semId(id)
     QWSSignalHandler::instance()->addWSLock(this);
 #endif
 
+    owned = false;
+
 #ifndef QT_POSIX_IPC
     if (semId == -1) {
         semId = semget(IPC_PRIVATE, 3, IPC_CREAT | 0666);
+        owned = true;
         if (semId == -1) {
             perror("QWSLock::QWSLock");
             qFatal("Unable to create semaphore");
@@ -100,7 +104,6 @@ QWSLock::QWSLock(int id) : semId(id)
     }
 #else
     sems[0] = sems[1] = sems[2] = SEM_FAILED;
-    owned = false;
 
     if (semId == -1) {
         // ### generate really unique IDs
@@ -134,9 +137,11 @@ QWSLock::~QWSLock()
 
     if (semId != -1) {
 #ifndef QT_POSIX_IPC
-        qt_semun semval;
-        semval.val = 0;
-        semctl(semId, 0, IPC_RMID, semval);
+	if (owned) {
+	    qt_semun semval;
+	    semval.val = 0;
+	    semctl(semId, 0, IPC_RMID, semval);
+	}
         semId = -1;
 #else
         // emulate the SEM_UNDO behavior for the BackingStore lock
diff --git a/src/gui/embedded/qwslock_p.h b/src/gui/embedded/qwslock_p.h
index d324e4f..d867d20 100644
--- a/src/gui/embedded/qwslock_p.h
+++ b/src/gui/embedded/qwslock_p.h
@@ -86,8 +86,8 @@ private:
     int lockCount[2];
 #ifdef QT_POSIX_IPC
     sem_t *sems[3];
-    bool owned;
 #endif
+    bool owned;
 };
 
 QT_END_NAMESPACE
```

Tony Liu

2017-5-2, Shenzhen