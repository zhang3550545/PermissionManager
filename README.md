# PermissionManager
android 6.0权限兼容库

### PermissionManager是一个对android6.0运行时申请权限的操作的封装，将重复调用的多个方法组合封装在一起，方便调用，简化代码和逻辑。PermissionManager的使用比较简单。大致就一下几个方法：

- execute(Activity activity,String permission)
    - 解释：方法只需要传递Activity和permission2个参数，会执行完检查权限和请求权限的步骤。
- execute(Activity activity,String... permissions)
    - 解释：这个方法可以一次传递多个权限，进行申请。
- executeDialog(Activity activity,String permission,Builder builder)
    - 解释:这个方法是在权限拒绝后，再次进行申请时会弹出一个提示的dialog，给用户一个提示或解释。Builder参数是PermissionManager的一个内部类，目的是通过builder对象携带Dialog需要的数据。
- onRequestPermissionsResult(int requestCode,String[] permissions, int[] grantResults)
    - 解释：这个方法主要是在将Activity中的onRequestPermissionsResult回调方法中的参数传递到PermissionManager中，进行处理。
- getGrantedInfo(String permission)
    - 解释：通过传递权限来判断是否授权,返回值boolean


eg:案例

在oncreate方法中调用initPermission()
```
private void initPermission() {
        //同时申请多个权限
//        PermissionManager.getInstance(getApplicationContext()).execute(this, Manifest.permission.RECORD_AUDIO, Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE);

        //请求单个，显示对话框的方式
        PermissionManager.getInstance(getApplicationContext()).executeDialog(this, Manifest.permission.RECORD_AUDIO,
                PermissionManager.getInstance(getApplicationContext()).new Builder(this)
                        .setMessage("应用需要获取您的录音权限，是否授权？")
                        .setTitle(getString(R.string.app_name))
                        .setIcon(R.mipmap.ic_launcher)
                        .setOk("OK")
                        .setCancel("CANCEL"));
    }
```

在Activity的onRequestPermissionsResult()方法中
```
 @Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    PermissionManager.getInstance(getApplicationContext()).onRequestPermissionsResult(requestCode, permissions, grantResults);
}
```

在需要使用到权限的代码之前做判断，如：

```
if (PermissionManager.getInstance(getApplicationContext()).getGrantedInfo(Manifest.permission.RECORD_AUDIO) ) {
    Toast.makeText(FirstActivity.this, "录音权限已经获取", Toast.LENGTH_SHORT).show();
} else {
    Toast.makeText(FirstActivity.this, "你还没有获取录音权限", Toast.LENGTH_SHORT).show();
}
```

*注意：在execute()和getGrantedInfo()方法使用的时候时机需要把握对，在execute()执行后申请权限，在onRequestPermissionsResult()方法中才获取最新的权限信息，再做处理。如果在此之前调用getGrantedInfo()可能拿不到正确的结果*
