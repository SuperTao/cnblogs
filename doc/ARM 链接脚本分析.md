分析连接脚本的语法规则

```
/* ----------------------------------------------------------------------------
 * Memory linker description
 * ------------------------------------------------------------------------- */
MEMORY
{
  /* ROM区，只读， 起始地址0x00000000, 长度4K */
  ROM  (r) : ORIGIN = 0x00000000, LENGTH = 4K
  /* FLASH区， 读，写，可执行， 起始地址0x00100000, 长度380K*/
  FLASH (xrw) : ORIGIN = 0x00100000, LENGTH = 380K
  /* PRAM , 32K */
  PRAM (xrw) : ORIGIN = 0x00200000, LENGTH = 32K

  DRAM (xrw) : ORIGIN = 0x20000000, LENGTH = 24K
  DRAM_DSP (xrw) : ORIGIN = 0x20006000, LENGTH = 48K
  DRAM_BB (xrw) : ORIGIN = 0x20012000, LENGTH = 16K
}

/* ----------------------------------------------------------------------------
 * Stack related defines and provided variables
 * ------------------------------------------------------------------------- */
/* 计算stack的地址 */
__stack = ORIGIN(DRAM) + LENGTH(DRAM);
/* 全局变量__stack, c语言可以应用 */
PROVIDE ( __stack = __stack ) ;

/*
 * Default stack sizes.
 * These are used by the startup in order to allocate stacks
 * for the different modes.
 */

__Main_Stack_Size = 1024 ;
PROVIDE ( _Main_Stack_Size = __Main_Stack_Size ) ;

__Main_Stack_Limit = __stack  - __Main_Stack_Size ;
PROVIDE ( _Main_Stack_Limit = __Main_Stack_Limit ) ;

/* ----------------------------------------------------------------------------
 * Heap related defines and provided variables
 * ------------------------------------------------------------------------- */
PROVIDE ( __Heap_Begin__ = __noinit_end__ ) ;
PROVIDE ( __Heap_Limit__ = __stack - __Main_Stack_Size ) ;

/*
 * The entry point is informative, for debuggers and simulators,
 * since the Cortex-M vector points to it anyway.
 */
/* 指定可执行文件的起始代码段是Reset_Handler */
ENTRY(Reset_Handler)

/* 
 * As is the VTOR register, we refer to it in startup documentation
 */
__VTOR = 0xE000ED08;

/* ----------------------------------------------------------------------------
 * Section definitions
 * ------------------------------------------------------------------------- */
SECTIONS
{
    /*
     * For Cortex-M devices, the beginning of the startup code is stored in
     * the .interrupt_vector section, which goes to FLASH
     */
	 /* DEFINED判断括号内的__app_rom_start，是否在全局符号表内，并且定义了，是就返回1，否返回0
	  * 再根据DEFINED结果进行判断__rom_start的值，是__app_rom_start，还是ORIGIN(FLASH)
	  */
    __rom_start = DEFINED(__app_rom_start) ? __app_rom_start : ORIGIN(FLASH);
    /* 计算image的大小 */
	__image_size = __data_init__ + SIZEOF(.data) - __rom_start;
	/* .text代码段, 保存在FLASH中，FLASH起始地址0x0010000*/
    .text __rom_start :
    {
	    /* 四字节对齐 */
        . = ALIGN(4);
		/* 强制链接器保留一些特定的section */
        KEEP(*(.interrupt_vector))

        /*
         * This section is here to store the startup code immediately after
         * the interrupt vectors, as required by the program ROM.
         */
		/* 所有文件的reset段，都放在interrupt_vector段后面 */
        *(.reset)
        
        /*
         * FOTA BootLoader descriptor
         */
        *(.rodata.fota.image-size)
        KEEP(*(.rodata.fota.build-id))
        
        /*
         * FOTA version descriptor
         */
		 /* 所有的.rodata.boot.version section 放到.text section里面*/
        *(.rodata.boot.version)

        /* Pre-initialization Code */
        . = ALIGN(4); 
        PROVIDE_HIDDEN (__preinit_array_start__ = .);

        /* System initialization and the platform initialization (if present)
         * should be first */
        KEEP(*(.preinit_array_sysinit .preinit_array_sysinit.*))
        KEEP(*(.preinit_array_platform .preinit_array_platform.*))

        /* Pre-initialization functions (to be executed before C++
         * constructors are run) */
        KEEP(*(.preinit_array .preinit_array.*))

        PROVIDE_HIDDEN (__preinit_array_end__ = .);

        /* Initialization Code */
        . = ALIGN(4);
        PROVIDE_HIDDEN (__init_array_start__ = .);

        KEEP(*(SORT(.init_array.*)))
        KEEP(*(.init_array))

        PROVIDE_HIDDEN (__init_array_end__ = .);

        /*
         * The program code is stored in the .text section,
         * which goes to FLASH.
         */
        . = ALIGN(4);
		/* 所有的文件的.text section*/
        *(.text .text.*)            /* all remaining code */
		/* 所有文件的.rodata section */
        *(.rodata .rodata.*)        /* read-only data (constants) */

        . = ALIGN(4);
        __dsp_start__ = . ;
        KEEP(*(.dsp .dsp.*))        /* all remaining DSP code */
        __dsp_end__ = . ;
		/* 四字节对齐 */
        . = ALIGN(4);
	/* .text全部链接到FLASH里面,Flash是前面MEMORY里面定义的，起始地址是0x00100000 */
    } >FLASH

    /*
     * This address is used by the startup code to
     * initialize the .data section.
     */
    . = ALIGN(4);
	/* 将定位符号'.'的值赋给__data_init */
	/* 也就是说__data_init__放在.text段的后面 */
    __data_init__ = .;

    /* Place the SystemClock variable needed for CMSIS in a place that is
     * compatible with the ROM's placement of this variable so that the
     * variable can be used by CMSIS and the ROM's flash write libary */
    .systemclock (NOLOAD) :
    {
        . = ALIGN(4);
        KEEP(*(.systemclock))
		/* 保存到DRAM中 */
    } > DRAM

    /*
     * The initialized data section.
     * The program executes knowing that the data is in the RAM
     * but the loader puts the initial values in the FLASH (inidata).
     * It is one task of the startup to copy the initial values from
     * FLASH to RAM.
     */
	 /* 查看__app_ram_start是否定义，并给__ram_start赋值 */
    __ram_start = DEFINED(__app_ram_start) ? __app_ram_start : .;
	/* .data数据段 */
	/* AT表示加载地址或者存储地址,指程序编译之后存放的地址,一般在ROM或者FLASH中  */
	/* 运行的时候，从AT指定的地址__data_init__中赋值到_ram_start中运行 */
	/* 从FLASH中复制到RAM里面运行 */
    .data __ram_start : AT ( __data_init__ )
    {
        . = ALIGN(4);

        /* This is used by the startup code to initialize the .data section */
        __data_start__ = . ;
        *(.data_begin .data_begin.*)
        *(.data .data.*)
        *(.data_end .data_end.*)
        . = ALIGN(4);

        /* This is used by the startup code to initialize the .data section */
        __data_end__ = . ;
    } >DRAM

    /*
     * The uninitialized data section. NOLOAD is used to avoid
     * the "section `.bss' type changed to PROGBITS" warning
     */
	 /* .bss未初始化的数据段, 存放在DRAM中 */
    .bss (NOLOAD) :
    {
        . = ALIGN(4);
        __bss_start__ = .;		// 把__bss_start_段赋值当前位置，bss段的起始位置
        *(.bss_begin .bss_begin.*)	// 所有的bss_begin和bss_begin.*段放到bss里面

        *(.bss .bss.*)		// 所有文件的.bss和.bss.*段放在bss_begin，bss_begin.*后面
        *(COMMON)			// COMMON放在.bss，.bss.*后面

        *(.bss_end .bss_end.*)	// 所有文件的.bss_end, .bss_end.* 放在COMMON后面
        . = ALIGN(4);
        __bss_end__ = .;		// 把__bss_end__段赋值为当前位置，结束位置
    } >DRAM

    .noinit (NOLOAD) :
    {
        . = ALIGN(4);
        __noinit_start__ = .;

        *(.noinit .noinit.*)

         . = ALIGN(4) ;
        __noinit_end__ = .;
    } > DRAM
    
    /* Check if there is enough space to allocate the main stack */
    ._stack (NOLOAD) :
    {
        . = ALIGN(4);
        
        . = . + __Main_Stack_Size ;	// 计算栈的大小放到DRAM中
        
        . = ALIGN(4);
    } >DRAM
}
```