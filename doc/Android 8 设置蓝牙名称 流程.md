记录android 8设置蓝牙名称的流程。
```


packages/apps/Settings/src/com/android/settings/bluetooth/BluetoothDeviceRenamePreferenceController.java
显示更改框
    @Override
    public boolean handlePreferenceTreeClick(Preference preference) {
        if (PREF_KEY.equals(preference.getKey())) {
            mMetricsFeatureProvider.action(mContext,
                    MetricsProto.MetricsEvent.ACTION_BLUETOOTH_RENAME);
            LocalDeviceNameDialogFragment.newInstance()
                    .show(mFragment.getFragmentManager(), LocalDeviceNameDialogFragment.TAG);
            return true;
        }   

        return false;
    }
	
	

packages/apps/Settings/src/com/android/settings/bluetooth/LocalDeviceNameDialogFragment.java
	public class LocalDeviceNameDialogFragment extends BluetoothNameDialogFragment {
    public static final String TAG = "LocalAdapterName";
    private LocalBluetoothAdapter mLocalAdapter;
	
	//对话框的title, Rename this device
	    @Override
    protected int getDialogTitle() {
        return R.string.bluetooth_rename_device;
    }
	
更改确认按键在父类中实现。

abstract class BluetoothNameDialogFragment extends InstrumentedDialogFragment
        implements TextWatcher {
		
		    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        String deviceName = getDeviceName();
        if (savedInstanceState != null) {
            deviceName = savedInstanceState.getString(KEY_NAME, deviceName);
            mDeviceNameEdited = savedInstanceState.getBoolean(KEY_NAME_EDITED, false);
        }
		// 对话框的显示已经操作函数。
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity())
                .setTitle(getDialogTitle())
                .setView(createDialogView(deviceName))
                .setPositiveButton(R.string.bluetooth_rename_button, (dialog, which) -> {
                    setDeviceName(mDeviceNameView.getText().toString().trim());
                })
                .setNegativeButton(android.R.string.cancel, null);
        mAlertDialog = builder.create();
        mAlertDialog.getWindow().setSoftInputMode(
                WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE);

        return mAlertDialog;
    }
	
	
	设置名称函数在子类中实现。
packages/apps/Settings/src/com/android/settings/bluetooth/LocalDeviceNameDialogFragment.java
	protected void setDeviceName(String deviceName) {
          mLocalAdapter.setName(deviceName);
      }
	  
	  
frameworks\base\packages\SettingsLib\src\com\android\settingslib\bluetooth\LocalBluetoothAdapter.java
	  public void setName(String name) {
        mAdapter.setName(name);
    }
	
	
frameworks\base\core\java\android\bluetooth\BluetoothAdapter.java
	    @RequiresPermission(Manifest.permission.BLUETOOTH_ADMIN)
    public boolean setName(String name) {
        if (getState() != STATE_ON) return false;
        try {
            mServiceLock.readLock().lock();
            if (mService != null) return mService.setName(name);
        } catch (RemoteException e) {
            Log.e(TAG, "", e);
        } finally {
            mServiceLock.readLock().unlock();
        }
        return false;
    }
	
	
	
frameworks\base\core\java\android\bluetooth\IBluetooth.aidl
interface IBluetooth
{
    boolean isEnabled();
    int getState();
    boolean enable();
    boolean enableNoAutoConnect();
    boolean disable();

    String getAddress();
    ParcelUuid[] getUuids();
    boolean setName(in String name);
	



对应的服务端程序如下。
frameworks\base\services\core\java\com\android\server\BluetoothManagerService.java

   case MESSAGE_BLUETOOTH_SERVICE_CONNECTED:
                {
                    if (DBG) Slog.d(TAG,"MESSAGE_BLUETOOTH_SERVICE_CONNECTED: " + msg.arg1);

                    IBinder service = (IBinder) msg.obj;
                    try {
                        mBluetoothLock.writeLock().lock();
                        if (msg.arg1 == SERVICE_IBLUETOOTHGATT) {
							mBluetoothGatt = IBluetoothGatt.Stub
                                    .asInterface(Binder.allowBlocking(service));
                            onBluetoothGattServiceUp();
                            break;
                        } // else must be SERVICE_IBLUETOOTH

                        //Remove timeout
                        mHandler.removeMessages(MESSAGE_TIMEOUT_BIND);

                        mBinding = false;
                        mBluetoothBinder = service;
						// 服务端和客户端建立连接
                        mBluetooth = IBluetooth.Stub.asInterface(Binder.allowBlocking(service));

                        if (!isNameAndAddressSet()) {
                            Message getMsg = mHandler.obtainMessage(MESSAGE_GET_NAME_AND_ADDRESS);
                            mHandler.sendMessage(getMsg);
                            if (mGetNameAddressOnly) return;
                        }
						
// 客户端程序继承IBluetooth.Stub
packages\apps\Bluetooth\src\com\android\bluetooth\btservice\AdapterService.java
    private static class AdapterServiceBinder extends IBluetooth.Stub {
        private AdapterService mService;

        public String getName() {
            if ((Binder.getCallingUid() != Process.SYSTEM_UID) &&
                (!Utils.checkCaller())) {
                Log.w(TAG, "getName() - Not allowed for non-active user and non system user");
                return null;
            }

            AdapterService service = getService();
            if (service == null) return null;
            return service.getName();
        }
		// 设置名称
        public boolean setName(String name) {
            if (!Utils.checkCaller()) {
                Log.w(TAG, "setName() - Not allowed for non-active user");
                return false;
            }

            AdapterService service = getService();
            if (service == null) return false;
            return service.setName(name);
        }
		
			
     boolean setName(String name) {
        enforceCallingOrSelfPermission(BLUETOOTH_ADMIN_PERM,
                                       "Need BLUETOOTH ADMIN permission");

        return mAdapterProperties.setName(name);
    }
	
	
packages\apps\Bluetooth\src\com\android\bluetooth\btservice\AdapterProperties.java
	    boolean setName(String name) {
        synchronized (mObject) {
            if (name.length() > BLUETOOTH_NAME_MAX_LENGTH_BYTES)
                name =  name.substring(0, BLUETOOTH_NAME_MAX_LENGTH_BYTES);
            return mService.setAdapterPropertyNative(
                    AbstractionLayer.BT_PROPERTY_BDNAME, name.getBytes());
        }
    }
	
调用JNI	
packages\apps\Bluetooth\src\com\android\bluetooth\btservice\AdapterService.java
	native boolean setAdapterPropertyNative(int type, byte[] val);
	

packages\apps\Bluetooth\jni\com_android_bluetooth_btservice_AdapterService.cpp
static jboolean setAdapterPropertyNative(JNIEnv* env, jobject obj, jint type,
                                         jbyteArray value) {
  ALOGV("%s", __func__);

  if (!sBluetoothInterface) return JNI_FALSE;

  jbyte* val = env->GetByteArrayElements(value, NULL);
  bt_property_t prop;
  prop.type = (bt_property_type_t)type;
  prop.len = env->GetArrayLength(value);
  prop.val = val;

  int ret = sBluetoothInterface->set_adapter_property(&prop);
  env->ReleaseByteArrayElements(value, val, 0);

  return (ret == BT_STATUS_SUCCESS) ? JNI_TRUE : JNI_FALSE;
}
```
Liu Tao
2019-3-27