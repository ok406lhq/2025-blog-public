## 优量汇广告播放失败问题记录

*最近我的红米手机更新了系统，由原来的Android 15升级到了Android 16，竟发现我们公司产品接入的腾讯广告（优量汇）无法播放的问题，在为期一周跟腾讯技术客服对线后，得到了问题的解决方案。*

![](/blogs/ad_issues/b17c03dd-361d-45f2-83fa-7fce1418526f.png)


---
先把报错贴出来吧：
``` java
gdt_ad_mob E  插件加载错误，回退到内置版本
gdt_ad_mob E  Fail to getfactory implement instance for interface:com.qq.e.comm.pi.POFactory (Ask Gemini)
	  com.qq.e.comm.managers.plugin.e: Fail to getfactory implement instance for interface:com.qq.e.comm.pi.POFactory
		at com.qq.e.comm.managers.plugin.PM.getFactory(Unknown Source:141)
		at com.qq.e.comm.managers.plugin.PM.getPOFactory(SourceFile:2)
		at com.qq.e.comm.managers.a$b.run(Unknown Source:9)
		at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:524)
		at java.util.concurrent.FutureTask.run(FutureTask.java:317)
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1302)
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:677)
		at java.lang.Thread.run(Thread.java:1119)
	  Caused by: java.lang.reflect.InvocationTargetException
		at java.lang.reflect.Method.invoke(Native Method)
		at com.qq.e.comm.managers.plugin.PM.getFactory(Unknown Source:79)
		at com.qq.e.comm.managers.plugin.PM.getPOFactory(SourceFile:2) 
		at com.qq.e.comm.managers.a$b.run(Unknown Source:9) 
		at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:524) 
		at java.util.concurrent.FutureTask.run(FutureTask.java:317) 
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1302) 
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:677) 
		at java.lang.Thread.run(Thread.java:1119) 
	  Caused by: java.lang.ExceptionInInitializerError
		at com.qq.e.comm.plugin.POFactoryImpl.getInstance(Unknown Source:12)
		at java.lang.reflect.Method.invoke(Native Method) 
		at com.qq.e.comm.managers.plugin.PM.getFactory(Unknown Source:79) 
		at com.qq.e.comm.managers.plugin.PM.getPOFactory(SourceFile:2) 
		at com.qq.e.comm.managers.a$b.run(Unknown Source:9) 
		at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:524) 
		at java.util.concurrent.FutureTask.run(FutureTask.java:317) 
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1302) 
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:677) 
		at java.lang.Thread.run(Thread.java:1119) 
	  Caused by: java.lang.UnsupportedOperationException: unsupport OS version
		at yaq.pro.<clinit>(pro.java:91)
		at com.qq.e.comm.plugin.POFactoryImpl.getInstance(Unknown Source:12) 
		at java.lang.reflect.Method.invoke(Native Method) 
		at com.qq.e.comm.managers.plugin.PM.getFactory(Unknown Source:79) 
		at com.qq.e.comm.managers.plugin.PM.getPOFactory(SourceFile:2) 
		at com.qq.e.comm.managers.a$b.run(Unknown Source:9) 
		at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:524) 
		at java.util.concurrent.FutureTask.run(FutureTask.java:317) 
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1302) 
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:677) 
		at java.lang.Thread.run(Thread.java:1119)
```
在排除了appid，广告位id的参数问题后，最终得出的结论就是接入的腾讯广告sdk版本暂未适配Android16的设备，需要更新到最近版本即可，也就是这个版本:

![](/blogs/ad_issues/32e8e295-66c1-4c5d-8809-432811665b84.png)


问题解决，广告可以正常播放！

![](/blogs/ad_issues/qq20240218110757.gif)

![](/blogs/ad_issues/c8816a3e-0ca2-40a0-9012-234109d2e064.png)

