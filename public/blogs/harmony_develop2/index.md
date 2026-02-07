
对于我来说，接触鸿蒙的第一课就是在unity游戏项目中接入发行SDK。今年早些时候接触到这个需求的时候，对于HarmonyOS还知之甚少，几个月的接触下来，实在感叹于鸿蒙的发展和成长速度，接下来我会以实战案例来讲述这个过程。

---
## 一. 接入准备
1. 新建项目工程
![新建项目](/blogs/harmony_develop2/e5e82a3aca1f4349b3d4b99303804942.png)
2. 选择最新的compatiable SDK版本后，点击"Finish"。
![Next](/blogs/harmony_develop2/1659a6aad84243d3aec5e774945440d5.png)
3. 新建项目后的目录如下
![项目结构](/blogs/harmony_develop2/f07f2add975c4be5bf43230e42122ba3.png)
工程目录的结构在HarmonyOS开发者官网有描述，这里就不多做介绍：[ArkTS工程目录结构（Stage模型）](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-project-structure)

## 二. 接入配置
1. 导入sdk文件包
1）在项目entry目录文件夹内新建libs目录
2）将sdk文件中的*.har 放入到entry
3）找到 entry 目录文件 oh-package.json5	，在 oh-package.json5 文件 dependencies 中引入 sdk 静态库文件，如图所示：
![导入har](/blogs/harmony_develop2/cbd0b16b22e647c380dc809c5594f59c.png)
需要注意的是，在以上配置"dependencies"的数据键名"xunyouhar"并不是自定义的，是引用该三方包所使用的依赖名称，建议与三方包包名，即三方包的oh-package.json5文件中的name字段保持一致。 这个可以参考官方文档：[引用共享包](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-har-import-V5)
4）最后在Terminal中运行命令："ohpm install"
![执行脚本](/blogs/harmony_develop2/a5c06426227a4a77b038994fbcd5681b.png)

2. 参数配置


1）需要找到entry模块里 src/main/module.json5 文件
```
{
  "name": "client_id",
  "value": "xxxxxxxx"
},
{
  "name": "app_id",
  "value": "xxxxxxx"  // 配置为为前面步骤中获取的APP ID
}
```
2）配置APP所需权限
配置app所需的必要权限，以下权限都必须配置
ohos.permission.INTERNET  网络权限
ohos.permission.ACCESS_NOTIFICATION_POLICY  通知消息权限
ohos.permission.GET_NETWORK_INFO 网络信息权限
ohos.permission.APP_TRACKING_CONSENT  获取设备标识权限
```
"requestPermissions": [
  {
    "name": "ohos.permission.INTERNET"
  },
  {
    "name": "ohos.permission.GET_NETWORK_INFO"
  },
  {
    "name": "ohos.permission.ACCESS_NOTIFICATION_POLICY"
  },
  {
    "name": "ohos.permission.APP_TRACKING_CONSENT",
    "reason": "$string:permission_APP_TRACKING_CONSENT",
    "usedScene": {
      "abilities": [
        "EntryAbility"
      ]
    },
  },
],
```
详细配置如图：	
![配置](/blogs/harmony_develop2/6b7b38c32c0e4f4993446ef4a1557423.png)
在 华为后台AppGallery Connect的"开发与服务"中，选择对应的项目或应用，在“常规 > 应用 ”下，找到应用的Client ID和APP ID。
![华为后台配置](/blogs/harmony_develop2/72284cdbac654ba987206b0714c23666.png)
3）配置应用app信息
找到根目录AppScope目录中app.json5，详细信息如下：
![app信息](/blogs/harmony_develop2/bd6299d3c68446bd9129780e0ba87e0e.png)
4）添加签名信息
在项目根目录下找到build-profile.json5 文件配置
![签名信息](/blogs/harmony_develop2/39da24456e2a44138c9d47bf7a25e655.png)
我这里用的是调试版本的签名，可以在"File > Project Structure... > Project > Signing Configs"中进行设置
![在这里插入图片描述](/blogs/harmony_develop2/1a47ce10de8a4a1aaafe24abe5425fbc.png)
![在这里插入图片描述](/blogs/harmony_develop2/34ff97c44e994f6e83805201154ec12b.png)
参考此文档，[调试签名配置](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-signing)

## 三. 接口说明
完成了项目工程的接入配置，后面就是代码层面的对接了，由于涉及到部分未经允许的公开代码，后边就不做太过详细的说明了。
接下来跟Android的接入流程相似，需要处理类似Application的启动接口，生命周期方法和游戏接入SDK基础的登录、支付、上报等通用接口。比如生命周期：
```
aboutToAppear() {
  XYGameSDK.getInstance().aboutToAppear()
}

onPageShow?() {
  XYGameSDK.getInstance().onPageShow()
  globalThis.ctx = this.getUIContext()
}

aboutToDisappear?() {
  XYGameSDK.getInstance().aboutToDisappear()
}

onPageHide?() {
  XYGameSDK.getInstance().onPageHide()
}

onBackPress?() {
  XYGameSDK.getInstance().onBackPress()
  return true;
}
```
登录接口：`` XYGameSDK.getInstance().login()`` 等等

**关于Unity跟Harmony的交互**
首先需要C#侧编写挂载脚本，参考资料：[Unity(团结引擎)与 HarmonyOS 通信 
](https://www.cnblogs.com/qiyer/p/18361436)
Arkts侧需要编写回调接收的代码，这边sdk在鸿蒙中没法用反射，只能在unity转化后的工程项目中，编写setOnCallbackInfo这个方法：
```
import { GlobalCallBack, XYGameSDK, } from 'xunyouhar';
import tuanjie from 'libtuanjie.so';

// 对发送给Unity团结引擎的接口进行二次封装
function setOnCallbackInfo(objectName: string, methodName: string) {
  let globalcallback: GlobalCallBack = {
    callback: (data: string) => {
      console.log("[xy][callback] --- > " + data)
      tuanjie.TuanjieSendMessage(objectName, methodName, data);
    }
  }
  XYGameSDK.getInstance().setGlobalCallBack(globalcallback)
}
```
**小插曲**
开发过程也遇到过一些问题，比如在ts语法中，定义对象是可以这样子做的：
![ts](/blogs/harmony_develop2/060d989a694c44838d53b2708ec5bb67.png)
但是Arkts语法中检查编译会报错，原因是在 ArkTS 语法中，需要显式标注对象字面量的类型，否则将导致编译时错误。在某些场景下，编译器可以根据上下文推断出字面量的类型。
最规范的做法是先定义一个接口（Interface）或类（Class）来描述数据的结构。
```
// 1. 首先定义一个接口，明确描述对象的结构
interface MyData {
  key: string; // 指定属性名和类型
  // 可以定义其他属性，如 value?: number; （问号表示可选属性）
}

// 2. 在声明变量时使用这个接口作为类型
let data: MyData = {
  key: 'value'
}; // 现在正确了
```
这个问题，也不算问题，其实就是typescript和Arkts语法风格相似的情况下，Arkrs更加强化了静态检查。

## 四. 总结

经过这段时间对HarmonyOS的实战开发，感叹于HarmonyOS对于unity游戏的适配和SDK的接入流程的顺畅，这个过程中我还学到了许多令人惊艳的特性，比如鸿蒙的“碰一碰”功能，以及多种富有创意的互动卡片。这些功能不仅在体验上非常新颖，也在设计理念上与Android系统有明显不同，体现出很多独特的创新。另外HarmonyOS的社区氛围非常浓厚，无论是开发者还是用户都积极参与其中，大家乐于分享知识、交流经验，共同推动生态的发展，这种蓬勃的社区文化正是鸿蒙生命力的重要体现。

ps. HarmonyOS作为新兴的操作系统，正致力于打造跟Android和IOS不同的软件生态，这个过程我认为是持久的，需要付出巨大艰辛工作的。相信未来的HarmonyOS会更加系统全面，加油鸿蒙！一起成长！~
