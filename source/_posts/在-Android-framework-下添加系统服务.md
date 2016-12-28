---
title: 在 Android framework 下添加系统服务
date: 2016-12-28 13:51:04
tags: 
  - Android
  - Framework
categories:
  - Android	
---


操作环境:  
系统：Linux (Ubuntu 14.04)  
平台：freescale  
Android版本：5.1

## 一、原理  

Android自身带有很多系统服务。这些系统服务都是在SystemServer类中被添加到ServiceManager中，继而进行管理。系统服务都是AIDL的实现类，也就是说调用系统服务其实都是通过AIDL进行跨进程通讯。

<!-- more -->
## 二、案例

### 2.1 创建AIDL接口文件。  
路径：`frameworks/base/core/java/android/service/aplex/IAplexGPSManager.aidl`。

```java  
package android.service.aplex;  
  
interface IAplexGPSService {  
    booblean setTimeByGPS();  
}  
```

### 2.2 将创建的AIDL加入编译中。  
#### 2.2.1 在`frameworks/base/Android.mk`添加  

	LOCAL_SRC_FILES += \  
	...  
	core/java/android/service/aplex/IAplexGPSManager.aidl \  
	...
	

#### 2.2.2 编译framework  

	source build/envsetup.sh
	
	lunch sabresd_6dq-user
	
	mmm framework/base/core/res/
	
	source build/envsetup.sh
	
	lunch sabresd_6dq-user
	
	mmm framework/base


### 2.3 创建新的系统服务类	
#### 2.3.1 创建`frameworks/base/core/java/android/service/aplex/AplexGPSManagerService.java`

```java
package android.service.aplex;

import android.content.Context;
import android.os.RemoteException;

public class AplexGPSManagerService extends IAplexGPSManager.Stub{
	private static final String TAG = "AplexGPSManagerService";  
	  
    private Context mContext;  
  
    public AplexGPSManagerService(Context context) {  
        mContext = context;  
    } 
	@Override
	public boolean setTimeByGPS() throws RemoteException {
		// TODO Auto-generated method stub
		return false;
	}

}

```

### 2.4 创建系统服务公开接口管理类  
路径：`frameworks/base/core/java/android/service/aplex/AplexGPSManager.java`
```java
package android.service.aplex;

import android.os.RemoteException;
import android.service.aplex.IAplexGPSManager;
public class AplexGPSManager {
	private static final String TAG = "AplexGPSManager";  
	  
    private IAplexGPSManager mService;  
  
    public AplexGPSManager(IAplexGPSManager service) {  
        mService = service;  
    }  
    public boolean setTimeByGPS() throws RemoteException {  
        return mService.setTimeByGPS();  
    }  
}

```
>Android系统中很多系统服务API没有全部公开，都是在Frameworks层通过Manager类封装一层来进行管理公开接口。也就是说对Application层公开都是通过Manager封装来决定的，这里我只公开了sayHello()。priFunction()是我故意不公开的，方便理解

### 2.5 将创建的服务添加进ServiceManager

路径：`frameworks/base/services/java/com/android/server/SystemServer.java`

```java
private void startOtherServices() {  
    ...  
    AudioService audioService = null;  
    AplexGPSManagerService mAplexgpsService = null;  
    ...  
    if (!disableMedia && !"0".equals(SystemProperties.get("system_init.startaudioservice"))) {  
        try {  
            Slog.i(TAG, "Audio Service");  
            audioService = new AudioService(context);  
            ServiceManager.addService(Context.AUDIO_SERVICE, audioService);  
        } catch (Throwable e) {  
            reportWtf("starting Audio Service", e);  
        }  
    }  
    ...  
    try {  
        Slog.i(TAG, "AplexGPSManager Service");  
        mAplexgpsService = new AplexGPSManagerService(context);  
        ServiceManager.addService(Context.APLEXGPS_SERVICE, mAplexgpsService);  
    } catch (Throwable e) {  
        reportWtf("starting AplexGPSManager", e);  
    }  
    ...  
}  
```
### 2.6 代码中的Context.APLEXGPS_SERVICE服务常量添加
路径：`frameworks/base/core/java/android/content/Context.java`

``` 
/** 
* Use with {@link #getSystemService} to retrieve a 
* {@link android.media.AudioManager} for handling management of volume, 
* ringer modes and audio routing. 
* 
* @see #getSystemService 
* @see android.media.AudioManager 
*/  
public static final String AUDIO_SERVICE = "audio";  
  
public static final String APLEXGPS_SERVICE = "aplexgps";  
```

### 2.7 注册服务
路径：frameworks/base/core/java/android/app/ContextImpl.java

```java
static {  
    ...  
    registerService(AUDIO_SERVICE, new ServiceFetcher() {  
        public Object createService(ContextImpl ctx) {  
            return new AudioManager(ctx);  
        }  
    });  
      
    registerService(APLEXGPS_SERVICE, new ServiceFetcher() {  
        public Object createService(ContextImpl ctx) {  
            IBinder iBinder = ServiceManager.getService(Context.APLEXGPS_SERVICE);  
                if (iBinder == null) {  
                    return null;  
                }  
                IAplexGPSManager service = IMAplexGPSManager.Stub  
                    .asInterface(iBinder);  
                return new AplexGPSManager(service);  
            }  
    });  
    ...  
}  
```

### 2.8 调用服务
```java
AplexGPSManager mAplexGPSManager = (AplexGPSManager) getSystemService(Context.APLEXGPS_SERVICE);  
try {  
    mAplexGPSManager.setTimeByGPS();  
} catch (RemoteException e) {  
    // TODO Auto-generated catch block  
    e.printStackTrace();  
}  
```

参考文章: http://blog.csdn.net/u012169524/article/details/51264979