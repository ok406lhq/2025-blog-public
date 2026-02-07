
当你第一次遇到一个 APK 包体超过 2GB 的时候，很多熟悉的反编译流程会在最关键的环节崩溃：zipalign、apksigner 这些看似“基础”的工具会报各种诡异的错误，甚至直接无法处理文件。原因并不是“你的 APK 有魔法”，而是工具和平台在设计时对文件尺寸和偏移有隐含假设（例如基于 32 位偏移、对 ZIP 中记录的长度有限制、以及 Google Play 的校验规则），在超大包体面前这些假设就失效了。

在这篇文章里，我会把我实际走通的那套可复现的方案给你讲清楚：为何会出错、如何定位问题、以及最直接、可靠的解决思路 —— 通过更换或回退到能够正确处理大文件的工具版本（zipalign.exe、apksigner.jar 等），并配合正确的签名与对齐顺序，最终让 APK 成功通过验证并可被反编译和安装。我会在文末把我验证过的工具版本和相应下载链接贴出来，方便大家复现。

接下来准备好你的超大 APK，我们开始吧。![冲](/blogs/decompile_issues/3dc37a75-eb8f-43bc-9f5c-1a3057bbe641.png)


---

### 复现步骤
很简单，准备好apk和工具，熟悉反编译的同学应该知道，当我们处理完apk，完成build apk的步骤后，会进行对齐（zipalign），再签名（apksigner）。
对齐需要用到工具zipalign.exe，命令如下：

```bash
 "F:\Temp\decompileOut\zipalign" -f 4 "F:/Temp/decompileOut/output.apk" "F:/Temp/decompileOut/output_aligned.apk" 
```

```bash
{zipalign.exe路径} -f 4 {准备好的apk路径}  {输出的apk路径}
```
当你敲下命令后你会看到：
![卢](/blogs/decompile_issues/d21ab6bc-2cf6-4c82-a397-79ccc8cce5e7.png)
具体原因可以参考这个博客： [zipalign在Windows平台处理大于2G apk问题](https://blog.csdn.net/liuyanggofurther/article/details/128902682)
总结就是：Windows 平台的 zipalign 在处理 >2GB APK 时因 ftell(long 为 32 位) 返回异常导致失败，解决思路是用 ftello/fseeko 与 off_t 替换并重编译生成可处理大文件的 zipalign.exe。这个重新生成的zipalign.exe我会放在文末。
通过更换zipalign工具成功对齐包体，接下来的步骤是签名。

```bash
F:/tool/PackToolsBate/tool/jre1.8/bin/java -jar F:/tool/PackToolsBate/tool/lib/apksigner.jar sign --ks F:/tool/PackToolsBate/config/keystore/agame-astar.keystore  --ks-key-alias agame-astar-idf --ks-pass pass:astar0731  --key-pass pass:astar0731  --out F:/Temp/decompileOut/output_signed.apk  F:/Temp/decompileOut/output_aligned.apk
```
```
{java路径}  -jar  {apksigner.jar路径} sign --ks  {keystore路径}   --ks-key-alias  {keystore别名}   --ks-pass pass:{keystore密码}   --key-pass pass:{key密码}  --out {输出的签名后的apk路径}  {未签名的apk路径}
```
可以成功签名，这一波没问题，但是在安装apk时会报错：
![子](/blogs/decompile_issues/d8b94b65-0cde-4c6c-aab9-174acf43f737.png)
还是因为包体大于2G的原因，接下来更换apksigner版本。我这里使用的是android sdk自带的"E:\Android\sdk\build-tools\30.0.0\lib"下的版本。
```
F:/tool/PackToolsBate/tool/jre1.8/bin/java  -jar F:/tool/PackToolsBate/tool/lib/apksigner-v30.jar sign --ks F:/tool/PackToolsBate/config/keystore/agame-astar.keystore  --ks-key-alias agame-astar-idf --ks-pass pass:astar0731  --key-pass pass:astar0731  --out F:/Temp/decompileOut/output_signed.apk  F:/Temp/decompileOut/output_aligned.apk
```
结果又有新的报错了：
![是](/blogs/decompile_issues/2c91a370-1cae-41f4-8f6f-7faeb4f4aae4.png)
原因就是jdk版本不对应，修改下java版本，再试下：(我这里用java 17的)
![傻](/blogs/decompile_issues/c33ff868-02af-473d-b7ee-44d5a6f7eab3.png)
成功签名。重新安装下apk：
![逼](/blogs/decompile_issues/ede586d4-2aea-413b-ad76-3d2506e10ff7.png)
问题解决！

### 总结
一句话结论：遇到 >2GB 的 APK 时，使用修补/重编译的 zipalign（支持 ftello/off_t）并配合与之兼容的 apksigner 和正确的 Java 版本，按“先对齐、再签名、最后验证”的流程即可可靠处理并安装大包。

### 附件
1. zipalign.exe的资源绑定在[附件](https://github.com/ok406lhq/MoToolBox/blob/main/zipalign.exe)里了，也可以去[csdn](https://download.csdn.net/download/baidu_34928905/91913273)下。

2. jar包的资源下载地址为：
点击下载：[apksigner-v30.jar(大小429kb)](/blogs/decompile_issues/apksigner-v30.jar)
