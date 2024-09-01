使用C#写ui时，希望能够读取到键盘输入的事件，用于快捷键或者其他操作。本文记录操作创建key event的方法。

#### 参考链接：

　　http://csharp.net-informations.com/gui/key-press-cs.htm

　　http://net-informations.com/q/faq/arrowkeys.html

#### 创建事件

- 选中某个需要读取键盘输入事件的控件，例如button,richTextBox等。

- 选中properties->event之后，列表中都是可以设置的事件。

- 选中需要的事件，例如KeyPress，然后进行双击, vs就会创建事件并生成代码。

如下图所示：

![](http://images2015.cnblogs.com/blog/745188/201609/745188-20160915212124570-1636093374.png)


#### C#代码

vs自动创建事件的程序之后，在代码中添加自己的功能。

经过测试，ascii码能够获取到，连 control + c(ascii码3)都可以.

```
        // 每按下一个按键就会触发一次
       private void richTextBoxReceive_KeyPress(object sender, KeyPressEventArgs e)
        {
            // 回车键
            if (e.KeyChar == (char)13) // enter key  
            {
                serialPort.Write("\r\n");
                rtbOutgoing.Text = "";
            }
            // control + C
            else if (e.KeyChar == (char)3)
            {
                MessageBox.Show("You pressed control + c");
                //通过串口发送 control + c
                byte[] by = { 0x03 };
                serialPort.Write(by, 0, by.Length);
            }
            // 其他的按键触发的时间不进行处理
            else if (e.KeyChar < 32 || e.KeyChar > 126)
            {
                e.Handled = true; // ignores anything else outside printable ASCII range
            }
            else
            {
                //这里是一个字符一个字符的发送。
                serialPort.Write(e.KeyChar.ToString());
            }
        }
```

#### 方向键处理

properties的event中没有关于方向键的事件，只需要在代码中重载C#的ProcessCmdKey方法。

对于control + c组合键，采用(Keys.Control | Keys.C)的方式进行判断。如下：

```
    protected override bool ProcessCmdKey(ref Message msg, Keys keyData)
        {
            //capture up arrow key
            if (keyData == Keys.Up)
            {
                MessageBox.Show("You pressed Up arrow key");
                return true;
            }
            //capture down arrow key
            if (keyData == Keys.Down)
            {
                MessageBox.Show("You pressed Down arrow key");
                return true;
            }
            //capture left arrow key
            if (keyData == Keys.Left)
            {
                MessageBox.Show("You pressed Left arrow key");
                return true;
            }
            //capture right arrow key
            if (keyData == Keys.Right)
            {
                MessageBox.Show("You pressed Right arrow key");
                return true;
            }
            // control + C 
            // 从这里可以看出，ProcessCmdKey函数也能够对单个字母的事件进行处理。例如A-Z,a-z。
            if (keyData == (Keys.Control | Keys.C))
            {
                MessageBox.Show("You pressed control + c");
                return true;
            }
            
            return base.ProcessCmdKey(ref msg, keyData);
        }
```

Tony Liu

2016-9-15, Shenzhen