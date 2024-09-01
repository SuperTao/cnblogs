安卓系统会首先读取BTM_DEF_LOCAL_NAME的值，如果为空，就使用"ro.product.model"作为蓝牙设备名。

system/bt/btif/src/btif_dm.cc
```
#define PROPERTY_PRODUCT_MODEL "ro.product.model"
......
static char* btif_get_default_local_name() {
  if (btif_default_local_name[0] == '\0') {
    int max_len = sizeof(btif_default_local_name) - 1; 
    if (BTM_DEF_LOCAL_NAME[0] != '\0') {
      strncpy(btif_default_local_name, BTM_DEF_LOCAL_NAME, max_len);
    } else {
      char prop_model[PROPERTY_VALUE_MAX];
      osi_property_get(PROPERTY_PRODUCT_MODEL, prop_model, ""); 
      strncpy(btif_default_local_name, prop_model, max_len);
    }    
    btif_default_local_name[max_len] = '\0';
  }
  return btif_default_local_name;
}
```

device/qcom/common/bdroid_buildcfg.h
```
#define BTM_DEF_LOCAL_NAME   "QCOM-BTD"
```

Tony Liu

2018-4-10