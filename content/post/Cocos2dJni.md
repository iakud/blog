---
categories:
- program
date: "2015-02-09T12:00:01Z"
title: Cocos2d通过Jni实现C++与Java相互调用
---

在Cocos2d项目中与运营平台(Java SDK)对接时使用了JNI。
<!--more-->
### 通过C++调用Java

在`JniUtil.h`文件中如下实现：
```cpp
#ifndef _JNIUTIL_H_
#define _JNIUTIL_H_

class JniUtil {
public:
	void static login(const char* zoneId, const char* zoneName);
};

#endif // _JNIUTIL_H_
```
在`JniUtil.cpp`文件中如下实现：
```cpp
#include "JniUtil.h"

#include <jni.h>
#include "platform/android/jni/JniHelper.h"

void JniUtil::login(const char* zoneId, const char* zoneName) {
	JniMethodInfo minfo;
	if (JniHelper::getStaticMethodInfo(minfo,
		"com/platform/test/JniUtil", "login",
		"(Ljava/lang/String;Ljava/lang/String;)V")) {

		jstring jzoneId = minfo.env->NewStringUTF(zoneId);
		jstring jzoneName = minfo.env->NewStringUTF(zoneName);
		minfo.env->CallStaticVoidMethod(minfo.classID,
			minfo.methodID, jzoneId, jzoneName);

		minfo.env->DeleteLocalRef(minfo.classID);
		minfo.env->DeleteLocalRef(jzoneId);
		minfo.env->DeleteLocalRef(jzoneName);
	}
}
```
Java的实现：
```java
package com.platform.test;

public class JniUtil {    
	private static void login(String zoneId, String zoneName) {
		// do
	}
}
```
### 通过Java调用C++

在Java的`JniUtil`类中定义一个方法，用于提供给Java调用C++：
```java
package com.platform.test;

public class JniUtil {
	public static native void onLogin(String result);
}
```
在`JniUtil.cpp`文件中如下实现：

方法名与Java类中的包名+方法名，以下划线连接
```cpp
extern "C" {
	void Java_com_platform_test_JniUtil_onLogin(JNIEnv* env,
		jobject thiz, jint jresult) {
		const char* result = env->GetStringUTFChars(jresult, NULL);
		CCLOG("onLogin : %s", result);
		env->ReleaseStringUTFChars(jresult, result);
	}
}
```