imx6dl i2c4 support

最近的项目用到了imx6dl的i2c4,其实完全可以用gpio-i2c的方法来实现。既然imx6的datasheet中提到有4个i2c，那么一定可以生成i2c的设备节点。

网上找了一些方法进行尝试，最终找到了一个patch，地址如下所示。

　　https://git.congatec.com/arm/qmx6_kernel/commit/593e5bd013eeaeda7405c587f851a9d12e9f8a75

　　https://community.nxp.com/thread/306096

将链接中的内容记录一下，方便以后查看。

连接中的提及到:
* imx6q有三个i2c和5个spi,
* imx6dl有四个i2c和4个spi.
* imx6dl的i2c4的时钟源来自pll3的ecspi_root.
		
代码更改操作方法(+表示添加，‘-’表示删除)：

#### arch/arm/mach-mx6/board-mx6q_sabresd.c

```
	imx6q_add_imx_i2c(0, &mx6q_sabresd_i2c_data);
 	imx6q_add_imx_i2c(1, &mx6q_sabresd_i2c_data);
 	imx6q_add_imx_i2c(2, &mx6q_sabresd_i2c_data);
	//添加i2c4的设备
+ 	if (cpu_is_mx6dl())
+ 		imx6q_add_imx_i2c(3, &mx6q_sabresd_i2c_data);
 	i2c_register_board_info(0, mxc_i2c0_board_info,
 			ARRAY_SIZE(mxc_i2c0_board_info));
 	i2c_register_board_info(1, mxc_i2c1_board_info,
```

#### arch/arm/mach-mx6/clock.c

```
 	_REGISTER_CLOCK("imx6q-ecspi.1", NULL, ecspi_clk[1]),
 	_REGISTER_CLOCK("imx6q-ecspi.2", NULL, ecspi_clk[2]),
 	_REGISTER_CLOCK("imx6q-ecspi.3", NULL, ecspi_clk[3]),
- 	_REGISTER_CLOCK("imx6q-ecspi.4", NULL, ecspi_clk[4]),
 	_REGISTER_CLOCK(NULL, "emi_slow_clk", emi_slow_clk),
 	_REGISTER_CLOCK(NULL, "emi_clk", emi_clk),
 	_REGISTER_CLOCK(NULL, "enfc_clk", enfc_clk),
... ...
};
	
+ static struct
+ clk_lookup imx6dl_i2c4 = _REGISTER_CLOCK("imx-i2c.3", NULL, ecspi_clk[4]);
+ static struct
+ clk_lookup imx6q_ecspi5 = _REGISTER_CLOCK("imx6q-ecspi.4", NULL, ecspi_clk[4]);
 
static void clk_tree_init(void)
{
...	...
 		clk_debug_register(lookups[i].clk);
 	}
 
 	/*
 	 * imx6q have 5 ecspi and 3 i2c
 	 * imx6dl have 4 ecspi and 4 i2c
 	 * imx6dl i2c4 use the imx6q ecspi5 clock source
 	 */
+ 	if (cpu_is_mx6dl()) {
+		clkdev_add(&imx6dl_i2c4);
+		clk_debug_register(imx6dl_i2c4.clk);
+	} else {
+		clkdev_add(&imx6q_ecspi5);
+		clk_debug_register(imx6q_ecspi5.clk);
+	}
```

#### arch/arm/plat-mxc/devices/platform-imx-i2c.c

```
const struct imx_imx_i2c_data imx6q_imx_i2c_data[] __initconst = {
#define imx6q_imx_i2c_data_entry(_id, _hwid)				\
 	imx_imx_i2c_data_entry(MX6Q, _id, _hwid, SZ_4K)
+ #define imx6dl_imx_i2c_data_entry(_id, _hwid)				\
+ 	imx_imx_i2c_data_entry(MX6DL, _id, _hwid, SZ_4K)
 	imx6q_imx_i2c_data_entry(0, 1),
 	imx6q_imx_i2c_data_entry(1, 2),
 	imx6q_imx_i2c_data_entry(2, 3),
+ 	imx6dl_imx_i2c_data_entry(3, 4),
 };
```
最后注意将对应的pin脚复用为i2c的功能。例如：

arch/arm/mach-mx6/board-mx6dl_sabresd.h

```
static iomux_v3_cfg_t mx6dl_sabresd_pads[] = {
    MX6DL_PAD_GPIO_7__I2C4_SCL,
    MX6DL_PAD_GPIO_8__I2C4_SDA,
```

Tony Liu

2016-11-19, Shenzhen