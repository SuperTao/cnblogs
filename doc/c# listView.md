使用listView时，需要设置单元格背景色。首先设置item.UseItemStyleForSubItems = false, 再通过BackColor来设置.

###### 参考：

http://www.liangshunet.com/ca/201309/586362553.htm

###### 代码

```
    listView1.GridLines = true;//表格是否显示网格线
　　listView1.FullRowSelect = true;//是否选中整行
　　listView1.View = View.Details;//设置显示方式
　　listView1.Scrollable = true;//是否自动显示滚动条
　　listView1.MultiSelect = true;//是否可以选择多行

    //添加首行的标题栏
    listView1.Columns.Add("NO", 60, HorizontalAlignment.Center);
    listView1.Columns.Add("uboot", 60, HorizontalAlignment.Center);
　　listView1.Columns.Add("kernel", 60, HorizontalAlignment.Center);
　　listView1.Columns.Add("rootfs", 60, HorizontalAlignment.Center);
　　listView1.Columns.Add("app", 60, HorizontalAlignment.Center);

    ListViewItem item = new ListViewItem();
    listView1.Items.Add(item);
　　item.SubItems.Clear();
    /* 设置subitem的Font, ForeColor, BackColor，为false时，只设置单个subitem,而不影响同一行的其他subitem */
    item.UseItemStyleForSubItems = false;
    item.SubItems[0].BackColor = Color.Red;
    //添加subitem
    item.SubItems.Add(new ListViewItem.ListViewSubItem());
    item.SubItems[1].BackColor = Color.Green;
    item.SubItems.Add(new ListViewItem.ListViewSubItem());
    item.SubItems[2].BackColor = Color.Blue;
    
    item.SubItems.Add(new ListViewItem.ListViewSubItem());
    item.SubItems[3].BackColor = Color.Yellow;

    item.SubItems.Add(new ListViewItem.ListViewSubItem());
    item.SubItems[4].BackColor = Color.Purple;
```

###### 效果图

![](http://images2015.cnblogs.com/blog/745188/201607/745188-20160704173117624-1369672750.png)

###### 删除所有行

```
    foreach(ListViewItem item  in listView1.Items)
    {
            listView1.Items.Remove(item);
    }
```

###### Author

Tony Liu

2016-7-4, Shenzhen