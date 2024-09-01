activity中数据的传递方式有2中，一种是使用putExtra()传递，另外一种是传递Bundle对象，使用putExtras()方法。

### 方法一

* 发送数据

putExtra()传送的是键值对，第一个参数是key,第二个参数是value。

```
	Intent intent = new Intent(MainActivity.this, Activity2.class);
	intent.putExtra("name", "hello");
	startActivity(intent);
```

* 接收数据

getString(): Extra根据key，获取value的值。

```
	Intent intent = getIntent();
	String name = intent.getStringExtra("name");
```

### 方法二

* 发送数据

使用bundle传输数据，putString(),putInt()，第一个参数key，第二个参数value.

可以传递String,int类型，或者其他。

```
	Intent i = new Intent(MainActivity.this, Activity3.class);
	Bundle bundle = new Bundle();
	bundle.putString("name", "hello");
	bundle.putInt("grade", 10);
	i.putExtras(bundle);
	startActivity(i);
```

* 接收数据

```
	Intent intent = getIntent();
	Bundle bundle = intent.getExtras();
	String name = bundle.getString("name");
	int grade = bundle.getInt("grade");
```
### Example

源码地址：https://github.com/TonySudo/ActivityData

设置2个按键分别用于测试2种方法传送的数据。在MainActivity之外再创建2个activity。

不同的按键启动不同的Activity，传递的方式也不同。

显示效果：

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170310131010592-629539894.png)

点击第一个按键

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170310131246279-316243204.png)


点击第二个按键

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170310131258936-104502569.png)

显示效果是一样的，只是内部传递的数据方式不同。


Tony Liu

2017-3-10, Shenzhen