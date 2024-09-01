
### 参考链接

http://www.h3c.com/cn/d_201708/1018838_30005_0.htm

### 漫游过程

例如STA目前工作在11信道，需要向6信道漫游。

首先切换到6信道进行认证。

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211002151803325-292991175.png)

#### Auth

STA发送Auth

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211002151249293-1317123290.png)


AP发送进行回复。

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211002151258964-153926096.png)


#### Reassoc request/response

接续切换回11信道进行工作。

然后跳到6信道进行重关联。

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211002152037998-1352578050.png)

* reassoc request

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211002152307793-1847179007.png)

* reassoc response

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211002152347435-1806614183.png)


#### 四次握手

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211002152426984-1005258344.png)
