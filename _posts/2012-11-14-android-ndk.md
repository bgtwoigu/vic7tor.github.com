---
layout: post
title: "Android NDK"
description: ""
category: 
tags: []
---
{% include JB/setup %}
#1.������װ
��װNDK��SDK��Ȼ��װeclipse��ADT���°��ADT��ʶ���NDK�ˡ���eclispe��ֱ�ӾͿ������ˣ�����װʲôCYGWIN��

#2.һ��ʵ��
##1.Android�е�����
��Ҫ����Native���������Ҫ�������������

    package com.example.ndk;

    public class MainActivity extends Activity {

        public native String getData(); ��Ϊ����
    
        static {
    	    System.loadLibrary("ndk");
        }
    }
##2.Native Code��ʵ��
�һ���Ŀ/Android Tools/Add Native Support��Ȼ���������֣����־���System.loadLibrary("ndk")�е�ndk�����ġ�

Ȼ�󣬵�ǰĿ¼�ͻ��һ��jni�ļ��С�ͬʱ����һ��cppԴ�����Android.mk�����Ū��CԴ�ļ����Ǹ�NewStringUTFʹ�û��б���ķ�������ʱ��֪����ô�����

cppԴ����Ϊ��

    #include <jni.h>

    extern "C" {
	jstring Java_com_example_ndk_MainActivity_getData(JNIEnv *env, jobject thisz);
    };

    jstring Java_com_example_ndk_MainActivity_getData(JNIEnv *env, jobject thisz)
    {
    	return env->NewStringUTF("Hello c, JNI");
    }
    
��Ϊ��cpp�ļ�����������extern���������������ӳɹ���.so�ļ��У��������е�Java_com_example_ndk_MainActivity_getData���ֲŻ���������������ʹ��c++���������⻯�����֡�

���ں�������Java_com_example_ndk_MainActivity_getData��Java�Ǳ���ġ�com_example_ndk�ǰ�����package com.example.ndk��MainActivity���ࡣgetData�Ǻ�������ע���Сд��

�����Ĳ�����ǰ�������ǹ̶��ģ�����ģ�����Java�е���������Ƿ��в�����������

�ο�ndk��Ŀ¼��sample������

����JAVA������C������ӳ������档

Java type	JNI type	C type		Stdint C type
boolean		Jboolean	unsigned char	uint8_t
byte		Jbyte 		signed char 	int8_t
char		Jchar		unsigned short	uint16_t
double		Jdouble		double		double
float		jfloat		float		float
int		jint		Int		int32_t
long		jlong		long long	int64_t
short		jshort		Short		int16_t

String���Ǳ�׼���ͣ������ú���NewStringUTF������

##native������java����
��StringҲ��һ���࣬jstring��jobject��typedef������Java���ൽ��native code�����Ϊjobject�����͡�������native codeʵ���У���Java��Ĳ������ĳ�jobject.

��������java�࣬Ҫ��native code�ı�ĵط�ʹ�ã���Ҫ����NewGlobalRef��

    JNIEXPORT void JNICALL Java_com_packtpub_Store_setColor
        (JNIEnv* pEnv, jobject pThis, jstring pKey, jobject pColor) {
        jobject lColor = (*pEnv)->NewGlobalRef(pEnv, pColor);
        if (lColor == NULL) {
            return;
        }
        StoreEntry* lEntry = allocateEntry(pEnv, &gStore, pKey);
        if (lEntry != NULL) {
            lEntry->mType = StoreType_Color;
            lEntry->mValue.mColor = lColor;
        } else {
            (*pEnv)->DeleteGlobalRef(pEnv, lColor);
        }
    }

NewGlobalRef��DeleteGlobalRef�ǳɶԳ��ֵġ�����NewGlobalRef��ԭ���Ƿ�ֹJava�������ռ����ɵ������������������ڱ�ĵط�ʹ�����������ô�㲻��Ҫ����NewGlobalRef��

##��native code���׳��쳣
����������

    public native String getString(String pKey)
        throws NotExistingKeyException, InvalidTypeException;

��native���׳��쳣��

	void throwNotExistingKeyException(JNIEnv* pEnv) {
		jclass lClass = (*pEnv)->FindClass(pEnv,
			��com/packtpub/exception/NotExistingKeyException��);
		if (lClass != NULL) {
			(*pEnv)->ThrowNew(pEnv, lClass, ��Key does not exist.��);
		}
		(*pEnv)->DeleteLocalRef(pEnv, lClass);
	}

##����
�õ�ʱ�ٲ�

#��native code�е���Java����
##1.Java��native code��ͬ��
һ��nativeʵ�ֵ�java����Ҫ�봴���̣߳�����native code��ʵ���У�ʹ��pthread�ӿڴ����߳̾����ˣ���pthread���еĺ����У�Ҫȡ��JavaVm������JavaVm::AttachCurrentThread��

������߳��У�����MonitorEnterһ��jobject��synchronized���ε�native�������ڶ��������ķ����ϡ�

##2.��native code�е���java����

