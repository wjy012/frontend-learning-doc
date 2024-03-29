
# 原生功能插件注册

在文件**res/xml/config.xml**中

```xml
<!-- 插件 begin -->
    <feature name="Audio">
        <param name="android-package" value="com.midea.web.plugin.AudioPlugin" />
    </feature>
    <feature name="MideaMap">
        <param
            name="android-package"
            value="com.midea.web.plugin.MideaMapPlugin" />
    </feature>
<!--
     <feature name="JS端的exec()的第三个参数">
        <param
            name="android-package"
            value="包路径com.midea.web.plugin.类名" />
    </feature>
-->
```



# CordovaPlugin

`execute` 方法用于响应js端的调用，每个Cordova插件都需要重写这个方法

```java
//class CordovaPlugin
public boolean execute(String action, String rawArgs, CallbackContext callbackContext) throws JSONException {
        Log.i("CordovaExec", "(String) " + serviceName + ":" + action);
        JSONArray args = new JSONArray(rawArgs);
        return execute(action, args, callbackContext);
    }
```

参数：

1. `String action`: 表示要执行的动作或命令的字符串。匹配js端`cordova.exec()`的第四个参数。
2. `JSONArray args`: 包含了来自JavaScript调用的参数的JSONArray。
3. `CallbackContext callbackContext`: 用于发送回调给JavaScript端。成功or错误回调

##### CallbackContext 常见用法：

**发送成功回调**

```java
callbackContext.success(); // 不发送任何消息
callbackContext.success("Operation completed successfully"); // 发送成功消息```

JSONObject json = new JSONObject();
json.put("text", text);
json.put("format", format);
callbackContext.success(json);
```



**发送失败回调**

```java
callbackContext.error("Operation failed"); // 发送错误消息
```

**发送PluginResult对象**
原生操作需要多次返回数据，或者长时间运行并提供进度更新

```java
PluginResult result = new PluginResult(PluginResult.Status.OK, "Progress update");
result.setKeepCallback(true); 	//通知Cordova继续保持这个回调链接打开，以便后续仍然可以发送数据。
callbackContext.sendPluginResult(result);
```

---

##### 以蓝牙插件`BluetoothAdapterPlugin`为例

```java
public class BluetoothAdapterPlugin extends CordovaPlugin {
    private CallbackContext scanCallbackContext;

    @Override
    public boolean execute(String action, JSONArray args,
                           CallbackContext callbackContext) throws JSONException {
        switch (action) {
            case "scanBLEDevices":
               //处理逻辑。。。
		scanCallbackContext = callbackContext;
		BTDeviceFinder.getFinder().setScanListener(object -> {
   		if (scanCallbackContext != null) {
        		PluginResult result =
                		new PluginResult(PluginResult.Status.OK, object);
        		result.setKeepCallback(true);
        		scanCallbackContext.sendPluginResult(result);
    		}
		});
		// 如果没有打开蓝牙设备，系统 会弹出窗子询问
		BTDeviceFinder.getFinder().openBTAdapter(cordova.getActivity());
		BTDeviceFinder.getFinder().scanNew(cordova.getActivity());
                return true;
	    case "connectBle":
                BTDeviceFinder.getFinder().connectBle(cordova.getActivity(), args);
                //蓝牙连接
                return true;
            case "stopScan":
                //停止扫描处理逻辑。。。
                callbackContext.success();
                return true;
            case "getBlueToothState":
                callbackContext.success(BTDeviceFinder.getFinder().getBlueToothState());
                return true;
           /**
			各种可能被调用的动作和对应处理逻辑...
			*/
        }
        return super.execute(action, args, callbackContext);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }
}
```
