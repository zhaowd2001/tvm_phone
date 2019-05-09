NFC on android 

   [NFC on android](https://github.com/zhaowd2001/tvm_phone/blob/master/android-nfc.md)


2019/5/9
  目录
  
[TOC]

## 代码下载 

   [Android NFC Demo](https://zwd.3wfocus.com/svn/files/trunk/tp/tvm/apps/android_rpc)

   ![NFC Demo](https://github.com/zhaowd2001/tvm_phone/blob/master/android-nfc.png?raw=true)
   
   
   
## 目的
   手头的门卡，想在android系统上刷出信息来。

## 代码
   网上搜了一下，用android 的 NFC 读卡模块， 能读到卡信息。
   参见 参考1.
   
## 申请NFC权限
   在工程的 AndroidManifest.xml 文件中添加如下代码，用于获取 NFC 硬件访问权限：
   
  ```
  <uses-permission android:name="android.permission.NFC" />
  <uses-permission android:name="android.permission.WAKE_LOCK" />
  <uses-feature android:name="android.hardware.nfc" android:required="true" />
  ```

## 为Activity 添加 singleTask
   仍然在是AndroidManifest.xml，为你的Activity添加singleTask.
   
   ```
   <activity android:name="your activity"
   android:launchMode="singleTask"/>
   ```

   不给 Activity 添加singleTask的话，每次你刷卡，当前的Activity会被 重新创建一遍，让你无法接收到android发给你的卡片信息。
   
## 接收卡片信息
   用户刷卡后，android系统会给app发通知。
   
   在app的Activity内，注册一下，就能收到刷卡时的通知。
   
###Activity.onCreate 内初始化NFC
   onCreate内，添加以下代码，向NFC系统注册一下，让刷卡后，NFC系统调用你:

   ```
       public boolean init(){
        NfcManager mNfcManager = (NfcManager) activity_.getSystemService(Context.NFC_SERVICE);
        mNfcAdapter = mNfcManager.getDefaultAdapter();
        if (mNfcAdapter == null) {
            message = "tv_nfc_notsupport";
            return  false;
        } else if ((mNfcAdapter != null) && (!mNfcAdapter.isEnabled())) {
            message = "tv_nfc_notwork";
            return  false;
        } else if ((mNfcAdapter != null) && (mNfcAdapter.isEnabled())) {
            message = "tv_nfc_working";
        }
        Class c = activity_.getClass();
        mPendingIntent =PendingIntent.getActivity(activity_, 0, new Intent(
                activity_, c), 0);

        init_NFC();
        return true;
    }

    private void init_NFC() {
        IntentFilter tagDetected = new IntentFilter(NfcAdapter.ACTION_TECH_DISCOVERED);
        tagDetected.addCategory(Intent.CATEGORY_DEFAULT);
    }
   ```
  
### Activity.onResume内处理一下NFC
   
   ```
       public  void resume(){
        if (mNfcAdapter != null) {
            mNfcAdapter.enableForegroundDispatch(activity_, mPendingIntent, null, null);
            Intent i = activity_.getIntent();
            String i_a = i.getAction();
            if (NfcAdapter.ACTION_TECH_DISCOVERED.equals(i_a)) {
                String data = processIntent(activity_.getIntent());
            }
        }
    }
   ```

### Activity.onPause 内停止接收 NFC 
   
   ```
    public  void stopNFC_Listener() {
        if(mNfcAdapter!=null) {
            mNfcAdapter.disableForegroundDispatch(activity_);
        }
    }
   ```

### Activity.onNewIntent 内接收NFC发给你的卡片信息
   
   ```
    public String processIntent(Intent intent) {
        String data = null;
        Tag tag = intent.getParcelableExtra(NfcAdapter.EXTRA_TAG);
        String[] techList = tag.getTechList();
        byte[] ID = new byte[20];
        data = tag.toString();
        ID =  tag.getId();
        String UID = Utils.bytesToHexString(ID);
        String IDString = bytearray2Str(hexStringToBytes(UID.substring(2,UID.length())), 0, 4, 10);
        data += "\n\nUID:\n" + UID;
        data += "\n\nID:\n" + IDString;
        data += "\nData format:";
        for (String tech : techList) {
            data += "\n" + tech;
        }
        data += "\nwg26status:-->" + PosUtil.getWg26Status(Long.parseLong(IDString)) + "\n";
        data += "wg34status:-->" + PosUtil.getWg34Status(Long.parseLong(IDString)) + "\n";
        return data;
    }
   ```

## 结论
   要接收到刷卡信息，只要在AndroidManifest.xml内添加NFC权限，注意也要为你的Actvity添加 singleTask属性。

然后，在你的Activity的onCreate/onResume/onPause/onNewIntent内，添加NFC代码，就能接收到刷卡信息。
 

## 参考
  - 参考1: [Android NFC开发教程](http://c.biancheng.net/view/3202.html)
  
