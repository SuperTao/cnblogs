ubuntu 12.04 安装django时，提示缺少setuptools.

转载自：  

http://blog.csdn.net/xudongtiankong/article/details/8180737  

安装setuptools的方法:

1. wget -q http://peak.telecommunity.com/dist/ez_setup.py

2. wget  http://pypi.python.org/packages/2.4/s/setuptools/setuptools-0.6c8-py2.4.egg

3. Python ez_setup.py

注意:第三步的url是根据第二步得到的url.  

关键:是要准备好2个东西:ez_setup.py和setuptools-0.6c8-py2.4.egg.