```
CORE/HDD/src/wlan_hdd_main.c
模块初始化：
static int __init hdd_module_init ( void)
{
   return hdd_driver_init();
}

static int hdd_driver_init( void)
{
   VOS_STATUS status;
   v_CONTEXT_t pVosContext = NULL;
   struct device *dev = NULL;
   int ret_status = 0;
#ifdef HAVE_WCNSS_CAL_DOWNLOAD
   int max_retries = 0;
#endif
#ifdef HAVE_CBC_DONE
   int max_cbc_retries = 0;
#endif

#ifdef WLAN_LOGGING_SOCK_SVC_ENABLE
   wlan_logging_sock_init_svc();
#endif

   ENTER();

   vos_wake_lock_init(&wlan_wake_lock, "wlan");

   pr_info("%s: loading driver v%s\n", WLAN_MODULE_NAME,
           QWLAN_VERSIONSTR TIMER_MANAGER_STR MEMORY_DEBUG_STR);

#ifdef ANI_BUS_TYPE_PCI

   dev = wcnss_wlan_get_device();

#endif // ANI_BUS_TYPE_PCI

#ifdef ANI_BUS_TYPE_PLATFORM

#ifdef HAVE_WCNSS_CAL_DOWNLOAD
   /* wait until WCNSS driver downloads NV */
   while (!wcnss_device_ready() && 10 >= ++max_retries) {
       msleep(1000);
   }

   if (max_retries >= 10) {
      hddLog(VOS_TRACE_LEVEL_FATAL,"%s: WCNSS driver not ready", __func__);
      vos_wake_lock_destroy(&wlan_wake_lock);
#ifdef WLAN_LOGGING_SOCK_SVC_ENABLE
      wlan_logging_sock_deinit_svc();
#endif

      return -ENODEV;
   }
#endif

#ifdef HAVE_CBC_DONE
   while (!wcnss_cbc_complete() && 10 >= ++max_cbc_retries) {
       msleep(1000);
   }
   if (max_cbc_retries >= 10) {
      hddLog(VOS_TRACE_LEVEL_FATAL, "%s:CBC not completed", __func__);
   }
#endif

   dev = wcnss_wlan_get_device();
#endif // ANI_BUS_TYPE_PLATFORM


   do {
      if (NULL == dev) {
         hddLog(VOS_TRACE_LEVEL_FATAL, "%s: WLAN device not found!!",__func__);
         ret_status = -1;
         break;
   }

#ifdef TIMER_MANAGER
      vos_timer_manager_init();
#endif

      /* Preopen VOSS so that it is ready to start at least SAL */
      status = vos_preOpen(&pVosContext);

   if (!VOS_IS_STATUS_SUCCESS(status))
   {
         hddLog(VOS_TRACE_LEVEL_FATAL,"%s: Failed to preOpen VOSS", __func__);
         ret_status = -1;
         break;
   }

   hddTraceInit();
   hdd_register_debug_callback();

#ifndef MODULE
      /* For statically linked driver, call hdd_set_conparam to update curr_con_mode
       */
      hdd_set_conparam((v_UINT_t)con_mode);
#endif

      // Call our main init function
      if (hdd_wlan_startup(dev))
      {
         hddLog(VOS_TRACE_LEVEL_FATAL,"%s: WLAN Driver Initialization failed",
                __func__);
         vos_preClose( &pVosContext );
         ret_status = -1;
         break;
      }

   } while (0);

   if (0 != ret_status)
   {
#ifdef TIMER_MANAGER
      vos_timer_exit();
#endif
#ifdef MEMORY_DEBUG
      vos_mem_exit();
#endif
      vos_wake_lock_destroy(&wlan_wake_lock);
#ifdef WLAN_LOGGING_SOCK_SVC_ENABLE
      wlan_logging_sock_deinit_svc();
#endif

      pr_err("%s: driver load failure\n", WLAN_MODULE_NAME);
   }
   else
   {
      //Send WLAN UP indication to Nlink Service
      send_btc_nlink_msg(WLAN_MODULE_UP_IND, 0);

      pr_info("%s: driver loaded\n", WLAN_MODULE_NAME);
   }

   EXIT();

   return ret_status;
}

int hdd_wlan_startup(struct device *dev )
{
	....
   // Read and parse the qcom_cfg.ini file
   status = hdd_parse_config_ini( pHddCtx );
   if ( VOS_STATUS_SUCCESS != status )
   {
      hddLog(VOS_TRACE_LEVEL_FATAL, "%s: error parsing %s",
             __func__, WLAN_INI_FILE);
      goto err_config;
   }
#ifdef MEMORY_DEBUG
   if (pHddCtx->cfg_ini->IsMemoryDebugSupportEnabled)
      vos_mem_init();

   hddLog(VOS_TRACE_LEVEL_INFO, "%s: gEnableMemoryDebug=%d",
          __func__, pHddCtx->cfg_ini->IsMemoryDebugSupportEnabled);
#endif

...
	// 设置日志输出的等级
   // Update VOS trace levels based upon the cfg.ini
   hdd_vos_trace_enable(VOS_MODULE_ID_BAP,
                        pHddCtx->cfg_ini->vosTraceEnableBAP);
   hdd_vos_trace_enable(VOS_MODULE_ID_TL,
                        pHddCtx->cfg_ini->vosTraceEnableTL);
   hdd_vos_trace_enable(VOS_MODULE_ID_WDI,
                        pHddCtx->cfg_ini->vosTraceEnableWDI);
   hdd_vos_trace_enable(VOS_MODULE_ID_HDD,
                        pHddCtx->cfg_ini->vosTraceEnableHDD);
   hdd_vos_trace_enable(VOS_MODULE_ID_SME,
                        pHddCtx->cfg_ini->vosTraceEnableSME);
   hdd_vos_trace_enable(VOS_MODULE_ID_PE,
                        pHddCtx->cfg_ini->vosTraceEnablePE);
   hdd_vos_trace_enable(VOS_MODULE_ID_PMC,
                         pHddCtx->cfg_ini->vosTraceEnablePMC);
   hdd_vos_trace_enable(VOS_MODULE_ID_WDA,
                        pHddCtx->cfg_ini->vosTraceEnableWDA);
   hdd_vos_trace_enable(VOS_MODULE_ID_SYS,
                        pHddCtx->cfg_ini->vosTraceEnableSYS);
   hdd_vos_trace_enable(VOS_MODULE_ID_VOSS,
                        pHddCtx->cfg_ini->vosTraceEnableVOSS);
   hdd_vos_trace_enable(VOS_MODULE_ID_SAP,
                        pHddCtx->cfg_ini->vosTraceEnableSAP);
   hdd_vos_trace_enable(VOS_MODULE_ID_HDD_SOFTAP,
                        pHddCtx->cfg_ini->vosTraceEnableHDDSAP);

   // Update WDI trace levels based upon the cfg.ini
   hdd_wdi_trace_enable(eWLAN_MODULE_DAL,
                        pHddCtx->cfg_ini->wdiTraceEnableDAL);
   hdd_wdi_trace_enable(eWLAN_MODULE_DAL_CTRL,
                        pHddCtx->cfg_ini->wdiTraceEnableCTL);
   hdd_wdi_trace_enable(eWLAN_MODULE_DAL_DATA,
                        pHddCtx->cfg_ini->wdiTraceEnableDAT);
   hdd_wdi_trace_enable(eWLAN_MODULE_PAL,
                        pHddCtx->cfg_ini->wdiTraceEnablePAL);
	...

}

static void hdd_vos_trace_enable(VOS_MODULE_ID moduleId, v_U32_t bitmask)
{
   wpt_tracelevel level;

   /* if the bitmask is the default value, then a bitmask was not
      specified in cfg.ini, so leave the logging level alone (it
      will remain at the "compiled in" default value) */
   //	0xffff
   if (CFG_VOS_TRACE_ENABLE_DEFAULT == bitmask)
   {
      return;
   }

   /* a mask was specified.  start by disabling all logging */
   vos_trace_setValue(moduleId, VOS_TRACE_LEVEL_NONE, 0);

   /* now cycle through the bitmask until all "set" bits are serviced */
   level = VOS_TRACE_LEVEL_FATAL;
   while (0 != bitmask)
   {
      if (bitmask & 1)
      {
         vos_trace_setValue(moduleId, level, 1);
      }
      level++;
      bitmask >>= 1;
   }
}


CORE/HDD/src/wlan_hdd_cfg.c
VOS_STATUS hdd_parse_config_ini(hdd_context_t* pHddCtx)
{
   int status, i=0;
   /** Pointer for firmware image data */
   const struct firmware *fw = NULL;
   char *buffer, *line, *pTemp = NULL;
   size_t size;
   char *name, *value;
   /* cfgIniTable is static to avoid excess stack usage */
   //MAX_CFG_INI_ITEMS   512, 最多512条
   static tCfgIniEntry cfgIniTable[MAX_CFG_INI_ITEMS];
   VOS_STATUS vos_status = VOS_STATUS_SUCCESS;

   memset(cfgIniTable, 0, sizeof(cfgIniTable));

   status = request_firmware(&fw, WLAN_INI_FILE, pHddCtx->parent_dev);

   if(status)
   {
      hddLog(VOS_TRACE_LEVEL_FATAL, "%s: request_firmware failed %d",__func__, status);
      vos_status = VOS_STATUS_E_FAILURE;
      goto config_exit;
   }
   if(!fw || !fw->data || !fw->size)
   {
      hddLog(VOS_TRACE_LEVEL_FATAL, "%s: %s download failed",
             __func__, WLAN_INI_FILE);
      vos_status = VOS_STATUS_E_FAILURE;
      goto config_exit;
   }

   hddLog(VOS_TRACE_LEVEL_INFO , "%s: qcom_cfg.ini Size %zu", __func__, fw->size);

   buffer = (char*)vos_mem_vmalloc(fw->size);

   if(NULL == buffer) {
      hddLog(VOS_TRACE_LEVEL_FATAL, "%s: kmalloc failure",__func__);
      release_firmware(fw);
      return VOS_STATUS_E_FAILURE;
   }
   pTemp = buffer;

   vos_mem_copy((void*)buffer,(void *)fw->data, fw->size);
   size = fw->size;

   while (buffer != NULL)
   {
      /*
       * get_next_line function used to modify the \n and \r delimiter
       * to \0 before returning, without checking if it is over parsing the
       * source buffer. So changed the function not to modify the buffer
       * passed to it and letting the calling/caller function to take
       * care of the return pointer validation and modification of the buffer.
       * So validating if the return pointer is not greater than the end
       * buffer address and modifying the buffer value.
       */
      line = get_next_line(buffer, (pTemp + (fw->size-1)));
      if(line >= (pTemp + fw->size)) {
         hddLog(VOS_TRACE_LEVEL_FATAL, "%s: INI file seems to be corrupted",
                  __func__);
         vos_status = VOS_STATUS_E_FAILURE;
         goto config_exit;
      }
      else if(line) {
         *(line - 1) = '\0';
      }

      buffer = i_trim(buffer);

      hddLog(LOG1, "%s: item", buffer);
		// "#"注释不解读
      if(strlen((char*)buffer) == 0 || *buffer == '#')  {
         buffer = line;
         continue;
      }
	  // 匹配到END退出解读ini文件
      else if(strncmp(buffer, "END", 3) == 0 ) {
         break;
      }
      else
      {
         name = buffer;
         while(*buffer != '=' && *buffer != '\0')
            buffer++;
         if(*buffer != '\0') {
            *buffer++ = '\0';
            i_trim(name);
            if(strlen (name) != 0) {
               buffer = i_trim(buffer);
               if(strlen(buffer)>0) {
                  value = buffer;
                  while(!i_isspace(*buffer) && *buffer != '\0')
                     buffer++;
                  *buffer = '\0';
                  cfgIniTable[i].name= name;
                  cfgIniTable[i++].value= value;
                  if(i >= MAX_CFG_INI_ITEMS) {
                     hddLog(LOGE,"%s: Number of items in %s > %d",
                        __func__, WLAN_INI_FILE, MAX_CFG_INI_ITEMS);
                     break;
                  }
               }
            }
         }
      }
      buffer = line;
   }

   //Loop through the registry table and apply all these configs
   vos_status = hdd_apply_cfg_ini(pHddCtx, cfgIniTable, i);

config_exit:
   release_firmware(fw);
   vos_mem_vfree(pTemp);
   return vos_status;
}
```

2018-8-13, Shenzhen