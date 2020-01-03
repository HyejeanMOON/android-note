在应用中会有需要实现确认当前网络状态是否可用的功能。
这里所说的网络状态可用指的是wifi或者蜂窝网络中有一个是可用的。
本身代码不是很难，但是因为版本的问题需要加上版本的判断。

直接上代码。

```kotlin
private fun checkNetworkState(context: Context): Boolean {
        // 获取网络连接管理器
        val connectivityManager =
            context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        // 需要判断版本
        // 如果当前版本大于安卓6棉花糖
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            // 通过网络连接管理器获得活跃网络
            val network = connectivityManager.activeNetwork ?: return false
            val activeNetwork = connectivityManager.getNetworkCapabilities(network) ?: return false
            return when {
 // 是否有wifi网络
activeNetwork.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) -> {
                    true
                }
 // 是否有蜂窝网络
 activeNetwork.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR) -> {
                    true
                }
                else -> {
                    false
                }
            }
        } else {
           // 小于安卓6棉花糖
           // 其实框架并没有强行限制版本要求，只是下面的api已经被废弃，防止框架彻底移除该api而引起的崩溃，进行版本判断
           // 通过网络连接管理器获得当前活跃的网络信息
            val nwInfo = connectivityManager.activeNetworkInfo ?: return false
            // 返回当前是否有网络连接
            return nwInfo.isConnected
        }
    }
```

---
在API29中废弃了activeNetworkInfo的API，但同时引入了NetworkCallback。
通过该API可以进行网络的实时监听，
但是有一个不足点是只有APP启动时才能进行监听回调，
如果想在不启动状态下进行监听，只能使用Broadcast。

代码如下：
（该代码来自：[https://www.jianshu.com/p/66afbd05c9b9](https://www.jianshu.com/p/66afbd05c9b9)）

1. 设置callback
```kotlin
@Override
public void onAvailable(Network network) {
    super.onAvailable(network);
    Log.d(TAG, "onAvailable: 网络已连接");
}

@Override
public void onLost(Network network) {
    super.onLost(network);
    Log.e(TAG, "onLost: 网络已断开");
}

@Override
public void onCapabilitiesChanged(Network network, NetworkCapabilities networkCapabilities) {
    super.onCapabilitiesChanged(network, networkCapabilities);
    if (networkCapabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED)) {
        if (networkCapabilities.hasTransport(NetworkCapabilities.TRANSPORT_WIFI)) {
            Log.d(TAG, "onCapabilitiesChanged: 网络类型为wifi");
            post(NetType.WIFI);
        } else if (networkCapabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR)) {
            Log.d(TAG, "onCapabilitiesChanged: 蜂窝网络");
            post(NetType.CMWAP);
        } else {
            Log.d(TAG, "onCapabilitiesChanged: 其他网络");
            post(NetType.AUTO);
        }
    }
}
```

2. 注册NetworkCallbackImpl
```kotlin
NetworkCallbackImpl networkCallback = new NetworkCallbackImpl();
NetworkRequest.Builder builder = new NetworkRequest.Builder();
NetworkRequest request = builder.build();
ConnectivityManager connMgr = (ConnectivityManager) NetworkListener.getInstance().getContext().getSystemService(Context.CONNECTIVITY_SERVICE);
if (connMgr != null) {
    connMgr.registerNetworkCallback(request, networkCallback);
}
```