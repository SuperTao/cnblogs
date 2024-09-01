本文记录am335x运行reboot命令时，内核中运行过程。

　　　　　　　　　　　　　　　　　　　　　Tony Liu, 2016-6-8, Shenzhen

参考链接：
        http://blog.csdn.net/wavemcu/article/details/8544333
```
kernel/sys.c
void kernel_restart(char *cmd)
{
    kernel_restart_prepare(cmd);                               ---------------+
    if (!cmd)                                                                 |
        printk(KERN_EMERG "Restarting system.\n");                            |
    else                                                                      |
        printk(KERN_EMERG "Restarting system with command '%s'.\n", cmd);     |
    kmsg_dump(KMSG_DUMP_RESTART);                                             |
    machine_restart(cmd);                                 ----------------+   |
}                                                                         |   |
EXPORT_SYMBOL_GPL(kernel_restart);                                        |   |
                                                                          |   |
void kernel_restart_prepare(char *cmd)                     <--------------|---+
{                                                                         |
    blocking_notifier_call_chain(&reboot_notifier_list, SYS_RESTART, cmd);|
    system_state = SYSTEM_RESTART;                                        |
    usermodehelper_disable();                                             |
    device_shutdown();                                                    |
    syscore_shutdown();                                                   |
}                                                                         |
                                                                          |
void machine_restart(char *cmd)                        <------------------+
{
    machine_shutdown();                                      ------------+
                                                                         |
    arm_pm_restart(reboot_mode, cmd);                        --------------+
                                                                         | |
    /* Give a grace period for failure to restart of 1s */               | |
    mdelay(1000);                                                        | |
                                                                         | |
    /* Whoops - the platform was unable to reboot. Tell the user! */     | |
    printk("Reboot failed -- System halted\n");                          | |
    while (1);                                                           | |
}                                                                        | |
                                                                         | |
void machine_shutdown(void)                                 <------------+ |
{                                                                          |
#ifdef CONFIG_SMP                                                          |
    smp_send_stop();                                                       |
#endif                                                                     |
}                                                                          |
                                                                           |
void (*arm_pm_restart)(char str, const char *cmd) = arm_machine_restart; <-+
EXPORT_SYMBOL_GPL(arm_pm_restart);

void arm_machine_restart(char mode, const char *cmd)
{

    /* Flush the console to make sure all the relevant messages make it
     * out to the console drivers */
    arm_machine_flush_console();

    /* Disable interrupts first */
    local_irq_disable();
    local_fiq_disable();

    /* Call the architecture specific reboot code. */
    arch_reset(mode, cmd);                                           ------+
}                                                                          |
                                                                           |
                                                                           |
arch/arm/mach-omap2/prcm.c                                                 |
void (*arch_reset)(char, const char *) = omap_prcm_arch_reset;             |
                                                                           |
static void omap_prcm_arch_reset(char mode, const char *cmd)        <------+
{
    s16 prcm_offs = 0;
    unsigned int val;
    if (cpu_is_omap24xx()) {
        omap2xxx_clk_prepare_for_reboot();

        prcm_offs = WKUP_MOD;
    } else if (cpu_is_am33xx()) {
        prcm_offs = AM33XX_PRM_DEVICE_MOD;
        //这里设置的是冷启动的方式
        omap2_prm_set_mod_reg_bits(OMAP4430_RST_GLOBAL_COLD_SW_MASK,
                    prcm_offs, AM33XX_PRM_RSTCTRL_OFFSET);
        //热启动方式如下
        //omap2_prm_set_mod_reg_bits(OMAP4430_RST_GLOBAL_WARM_SW_MASK,
        //            prcm_offs, AM33XX_PRM_RSTCTRL_OFFSET);
    } else if (cpu_is_omap34xx()) {
        prcm_offs = OMAP3430_GR_MOD;
        omap3_ctrl_write_boot_mode((cmd ? (u8)*cmd : 0));
    } else if (cpu_is_omap44xx()) {
        omap4_prminst_global_warm_sw_reset(); /* never returns */
    } else {
        WARN_ON(1);
    }

    /*
     * As per Errata i520, in some cases, user will not be able to
     * access DDR memory after warm-reset.
     * This situation occurs while the warm-reset happens during a read
     * access to DDR memory. In that particular condition, DDR memory
     * does not respond to a corrupted read command due to the warm
     * reset occurrence but SDRC is waiting for read completion.
     * SDRC is not sensitive to the warm reset, but the interconnect is
     * reset on the fly, thus causing a misalignment between SDRC logic,
     * interconnect logic and DDR memory state.
     * WORKAROUND:
     * Steps to perform before a Warm reset is trigged:
     * 1. enable self-refresh on idle request
     * 2. put SDRC in idle
     * 3. wait until SDRC goes to idle
     * 4. generate SW reset (Global SW reset)
     *
     * Steps to be performed after warm reset occurs (in bootloader):
     * if HW warm reset is the source, apply below steps before any
     * accesses to SDRAM:
     * 1. Reset SMS and SDRC and wait till reset is complete
     * 2. Re-initialize SMS, SDRC and memory
     *
     * NOTE: Above work around is required only if arch reset is implemented
     * using Global SW reset(GLOBAL_SW_RST). DPLL3 reset does not need
     * the WA since it resets SDRC as well as part of cold reset.
     */

    /* XXX should be moved to some OMAP2/3 specific code */
    omap2_prm_set_mod_reg_bits(OMAP_RST_DPLL3_MASK, prcm_offs,
                   OMAP2_RM_RSTCTRL);
    omap2_prm_read_mod_reg(prcm_offs, OMAP2_RM_RSTCTRL); /* OCP barrier */
        val = omap2_prm_read_mod_reg(prcm_offs, AM33XX_PRM_RSTTIME_OFFSET);
        printk(KERN_ALERT "<<Tony>> reset time value 2: %x\n", val);
}
```