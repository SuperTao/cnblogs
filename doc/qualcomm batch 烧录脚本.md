在烧录android系统候用到了windows的批处理文件，拿出来分析一下，顺便记录一下高通平台烧录系统的命令。

```
@echo off						
:: @ ：不显示后面的命令，就是后面的”echo off“命令，
:: echo off ：在此语句后所有运行的命令都不显示命令行本身，默认是on
color 2f						
:: 设置背景色和前景色，都是16进制的，第一个数字2设置背景色，第二个数字f设置前景色
mode con cols=60 lines=30
:: 设置窗口60列，30行
adb reboot bootloader
title Tony Test
:: 窗口的标题

set AP_ROOT=\\tony\msm\out\target\product\msm
:: 设置文件放置的路径,set设置变量

:start
:: 设置一个名称是"start"标签，":"后面是标签
cls
:: 清屏
echo ----------------------------------------
echo    请选择你要进行的操作，然后按回车
echo ----------------------------------------
echo.
echo        1，完整升级     2，升级AP
echo        3, 升级MP       4，boot
echo        5，system       6，userdata
echo        7，recovery     8，splash
echo        9，cache        a，persist
echo        b，emmc         q，退出 
echo.
:: 输出一个"回车换行"，注意echo后面有一个.
:: 直接输出echo的话，会显示当前是echo off状态还是echo on状态
set /p n=      请选择：
:: set /p选项，用于读取用户输入，保存到n中。
if "%n%"=="1" (goto all_update) 
:: 判断用户输入
if "%n%"=="2" (goto ap_update)
if "%n%"=="3" (goto mp_update)
if "%n%"=="4" (goto boot_update)
if "%n%"=="5" (goto system_update)
if "%n%"=="6" (goto userdata_update)
if "%n%"=="7" (goto recovery_update)
if "%n%"=="8" (goto splash_update)
if "%n%"=="9" (goto cache_update)
if "%n%"=="a" (goto persist_update)
if "%n%"=="b" (goto emmc_update)
if "%n%"=="q" (goto updata_exit)

:all_update 
echo fastboot Partition...
fastboot flash partition %AP_ROOT%\gpt_main0.bin

echo fastboot MP....
fastboot flash modem %AP_ROOT%\NON-HLOS.bin
fastboot flash rpm %AP_ROOT%\rpm.mbn
fastboot flash sbl1 %AP_ROOT%\sbl1.mbn
fastboot flash tz %AP_ROOT%\tz.mbn
fastboot flash hyp %AP_ROOT%\hyp.mbn

echo fastboot AP....
fastboot flash boot %AP_ROOT%\boot.img
fastboot flash -S 200M system %AP_ROOT%\system.img
fastboot flash cache %AP_ROOT%\cache.img
fastboot flash persist %AP_ROOT%\persist.img
fastboot flash recovery %AP_ROOT%\recovery.img
fastboot flash splash %AP_ROOT%\splash.img
fastboot flash userdata %AP_ROOT%\userdata.img 
fastboot flash aboot %AP_ROOT%\emmc_appsboot.mbn
fastboot flash IPSM %AP_ROOT%\IPSM.img
fastboot flash oem %AP_ROOT%\oem.img


echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
goto start

:ap_update
echo fastboot AP....
fastboot flash boot %AP_ROOT%\boot.img
fastboot flash system %AP_ROOT%\system.img
fastboot flash cache %AP_ROOT%\cache.img
fastboot flash persist %AP_ROOT%\persist.img
fastboot flash recovery %AP_ROOT%\recovery.img
fastboot flash splash %AP_ROOT%\splash.img
fastboot flash userdata %AP_ROOT%\userdata.img 
fastboot flash aboot %AP_ROOT%\emmc_appsboot.mbn
fastboot flash oem %AP_ROOT%\oem.img
echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
:: pause暂停
goto start

:mp_update
echo fastboot MP....
fastboot flash modem %AP_ROOT%\NON-HLOS.bin
fastboot flash rpm %AP_ROOT%\rpm.mbn
fastboot flash sbl1 %AP_ROOT%\sbl1.mbn
fastboot flash dbi %AP_ROOT%\sdi.mbn
fastboot flash tz %AP_ROOT%\tz.mbn
fastboot flash hyp %AP_ROOT%\hyp.mbn
echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
goto start

:boot_update
fastboot flash boot %AP_ROOT%\boot.img
echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
goto start

:system_update
fastboot flash system %AP_ROOT%\system.img
echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
goto start

:userdata_update
fastboot flash userdata %AP_ROOT%\userdata.img
echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
goto start

:recovery_update
fastboot flash recovery %AP_ROOT%\recovery.img
echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
goto start

:splash_update
fastboot flash splash %AP_ROOT%\splash.img
echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
goto start

:cache_update
fastboot flash cache %AP_ROOT%\cache.img
echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
goto start

:persist_update
fastboot flash persist %AP_ROOT%\persist.img
echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
goto start

:emmc_update
fastboot flash aboot %AP_ROOT%\emmc_appsboot.mbn
echo --------------------------
echo     ++++++++OK++++++++
echo --------------------------
pause
goto start

:updata_exit
fastboot reboot
exit
```

显示效果

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213191216488-510291208.png)


Tony Liu

2017-12-13