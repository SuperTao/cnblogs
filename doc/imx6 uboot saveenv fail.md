uboot设置环境变量之后，不能保存在EMMC中，出现错误。

```
MX6SDL SABRESD U-Boot > saveenv
Saving Environment to SPI Flash... 
Environment SPI flash not initialized
```

板子上没有flash，使用EMMC。

查找出错地方，发现uboot调用的是uboot/common/env_sf.c中的saveenv函数。

查看uboot/common/Makefile
```
COBJS-$(CONFIG_ENV_IS_IN_ONENAND) += env_onenand.o
COBJS-$(CONFIG_ENV_IS_IN_SATA) += env_sata.o
COBJS-$(CONFIG_ENV_IS_IN_SPI_FLASH) += env_sf.o
COBJS-$(CONFIG_ENV_IS_IN_MMC) += env_mmc.o
```

查找对应的宏，位于uboot/include/configs/mx6dl_sabresd.h
更改保存在MMC中。
```
/* Monitor at beginning of flash */
//#define CONFIG_FSL_ENV_IN_SF
// Tony 2016-12-18
#define CONFIG_FSL_ENV_IN_MMC
//#define CONFIG_FSL_ENV_IN_NAND 
```

这样一来就调用env_mmc.c中的saveenv函数。

重新编译，保存环境变量，出现错误。

```
MX6SDL SABRESD U-Boot > saveenv
Saving Environment to MMC...
MMC Device 3 not found
No MMC card found
```

uboot中查看EMMC

```
MX6SDL SABRESD U-Boot > mmc list
FSL_USDHC: 0
FSL_USDHC: 1
FSL_USDHC: 2
```

应该从MMC2启动，而不是MMC3。查找报错的地方，进行更改。

uboot/common/env_mmc.c

```
int saveenv(void)
{
    struct mmc *mmc = find_mmc_device(mmc_env_devno);

    if (init_mmc_for_env(mmc))
        return 1;

    printf("Writing to MMC(%d)... ", mmc_env_devno);
    if (write_env(mmc, CONFIG_ENV_SIZE, CONFIG_ENV_OFFSET, env_ptr)) {
        puts("failed\n");
        return 1;
    }   

    puts("done\n");
    return 0;
}
```

mmc_env_devno变量赋值的地方。

uboot/common/env_mmc.c

```
int env_init(void)
{
    /* use default */
    gd->env_addr = (ulong)&default_environment[0];
    gd->env_valid = 1;

#ifdef CONFIG_DYNAMIC_MMC_DEVNO
    extern int get_mmc_env_devno(void);
    mmc_env_devno = get_mmc_env_devno();
#else
    mmc_env_devno = CONFIG_SYS_MMC_ENV_DEV;
#endif

    return 0;
}
```

CONFIG_DYNAMIC_MMC_DEVNO和CONFIG_SYS_MMC_ENV_DEV的定义在uboot/include/configs/mx6dl_sabresd.h

注释掉CONFIG_DYNAMIC_MMC_DEVNO，采用静态的CONFIG_SYS_MMC_ENV_DEV。

uboot/include/configs/mx6dl_sabresd.h
```
#ifdef CONFIG_CMD_MMC
    #define CONFIG_MMC
    #define CONFIG_GENERIC_MMC
    #define CONFIG_IMX_MMC
    #define CONFIG_SYS_FSL_USDHC_NUM        3
    #define CONFIG_SYS_FSL_ESDHC_ADDR       0
    #define CONFIG_SYS_MMC_ENV_DEV  2
    #define CONFIG_DOS_PARTITION    1
    #define CONFIG_CMD_FAT      1
    #define CONFIG_CMD_EXT2     1

    /* detect whether SD1, 2, 3, or 4 is boot device */
    // Tony 2016-12-18
//  #define CONFIG_DYNAMIC_MMC_DEVNO

    /* SD3 and SD4 are 8 bit */
    #define CONFIG_MMC_8BIT_PORTS   0xC
    /* Setup target delay in DDR mode for each SD port */
    #define CONFIG_GET_DDR_TARGET_DELAY
#endif
```

编译之后，再次保存，OK。

```
MX6SDL SABRESD U-Boot > saveenv
Saving Environment to MMC...
Writing to MMC(2)... done
```

Tony Liu

2016-12-18, Shenzhen