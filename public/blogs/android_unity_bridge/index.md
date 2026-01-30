## 问题记录 📝
Unity游戏在接入UC九游渠道（网游）S	DK后，调用支付弹窗后出现的画面移位的问题：

<video width="640" height="360" controls>
  <source src="/blogs/android_unity_bridge/9AF3CF64F79931AA8C3A1F6308461892.mp4" type="video/mp4">
  您的浏览器不支持HTML5视频播放
</video>

## 解决过程 🔍
### **方案A：在`onResume`中强制设置横屏**
  ```java
  @Override
  protected void onResume() {
      super.onResume();
      if(getRequestedOrientation() != ActivityInfo.SCREEN_ORIENTATION_SENSOR_LANDSCAPE){
          setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_SENSOR_LANDSCAPE);
      }
  }
  ```
**为何可能无效**：`onResume`时再设置方向可能为时已晚，窗口尺寸的变化可能已经发生或正在处理中，无法阻止因方向切换引发的底层View尺寸重绘。

###  **方案B：在`AndroidManifest.xml`中声明`screenOrientation`**
  ```xml
  <activity
      android:name=".YourGameActivity"
      android:screenOrientation="sensorLandscape" />
  ```
  **为何可能无效**：这个设置能很好地保证你的`Activity`在大部分情况下以横屏启动和运行。但当游戏需要跳转到另一个独立的、强制竖屏的第三方应用时（例如，通过SDK拉起支付宝App进行支付），这个设置就可能不够了。因为支付宝这类应用有自己的任务栈和窗口管理，它们会强制屏幕变为竖屏。当支付完成，从支付宝返回到你的游戏时，Android系统需要将屏幕方向从竖屏“切换回”横屏。在这个切换过程中，如果窗口尺寸信息没有被Unity的渲染引擎正确捕获和处理，就会导致渲染表面（Surface）的尺寸与触控区域不匹配，从而出现画面移位。

### 方案C：通过`onWindowFocusChanged`刷新视图

这个方案的核心思想是，在游戏窗口重新获得焦点时（即从支付页面返回后），强制进行一次UI刷新，期望能间接触发Unity的渲染表面（Surface）尺寸校正。

**操作步骤**：
在你的`UnityPlayerActivity`子类中，重写`onWindowFocusChanged`方法。

```java
import android.os.Handler;
import android.os.Looper;
import android.view.View;

// ... 在你的Activity类中

@Override
public void onWindowFocusChanged(boolean hasFocus) {
    super.onWindowFocusChanged(hasFocus);
    if (hasFocus) {
        // 当窗口重新获得焦点时，重新应用沉浸式模式
        setImmersiveMode();
    }
}

// 示例：一个设置沉浸式模式的方法
private void setImmersiveMode() {
    // 建议使用 post, 确保在UI线程消息队列的稍后阶段执行，避开一些时序问题
    getWindow().getDecorView().post(() -> {
        View decorView = getWindow().getDecorView();
        int flags = View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                  | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                  | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                  | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                  | View.SYSTEM_UI_FLAG_FULLSCREEN
                  | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY;
        decorView.setSystemUiVisibility(flags);
    });
}

// 同时，在 onResume 中也调用，作为双重保险
@Override
protected void onResume() {
    super.onResume();
    setImmersiveMode();
}
```

**为何可能无效：**  
-   **时机问题**：`postDelayed` 的延迟时间是固定的（例如500ms），但这只是一个猜测值。在性能差的设备上，窗口可能还未稳定；在性能好的设备上，你的操作可能又被系统后续的某个绘制操作给覆盖了。这是一个不稳定的“赌博式”修复。

-   **方法间接**：`setImmersiveMode()` 只是一个“暗示”，请求系统UI重新布局。它并不直接命令Unity的渲染表面（SurfaceView）去调整大小。如果系统认为没必要，这个暗示可能被完全忽略。

-   **SDK干扰**：你无法保证第三方SDK在退出时不会执行一些清理代码。很可能在你费力设置好屏幕状态后，SDK的代码紧接着又把它改回去了。

-   **事件不触发**：如果SDK拉起的页面是半透明的或是一个对话框（Dialog），你的游戏Activity可能根本没有完全失去窗口焦点（`Window Focus`），因此`onWindowFocusChanged`事件可能不会如预期般触发。

它试图“修复”一个已经发生的问题，因此，它是一种不可靠的、需要大量测试的备用方案。

### 方案D：在 Manifest 中处理配置变更
这是解决此类问题的最标准、最有效的方法。通过告诉Android系统，你的`Activity`将自行处理屏幕方向和尺寸的变化，可以阻止系统为了应用这些变化而销毁并重建你的`Activity`。

在`AndroidManifest.xml`中，为你游戏的`Activity`添加 `android:configChanges` 属性。

```xml
<activity
    android:name=".YourGameActivity"
    android:screenOrientation="sensorLandscape"
    android:configChanges="fontScale|layoutDirection|density|smallestScreenSize|screenSize|uiMode|screenLayout|orientation|navigation|keyboardHidden|keyboard|touchscreen|locale|mnc|mcc"
    android:exported="true">
    <!-- 其他intent-filter等配置 -->
</activity>
```
**结果还是无效😭** 

## 解决方案 ✔️
在详细对比了其他正常游戏的处理和配置，发现了区别！就是这个 "**resizeableactivity**" 属性的差异：
![](/blogs/android_unity_bridge/bce8f230192043ba93950929f0a20697.png)
如果移除了这个 **`resizeableactivity`** 属性或者设置为true，问题就可以解决了！~~ 🎉🎉🎉

**为何有效：**  推测当 Unity 游戏调用第三方支付/跳转到外部应用返回后出现画面移位，通常是因为 Activity 的窗口被“重置/重建”或被系统以不正确的方式 resize；将目标 Activity 的 android:resizeableActivity 设置为 "true"（或移除该属性，让系统用默认值）可以避免系统对 Activity 做导致 Surface/输入错位的重建或不当重排，从而解决问题。

![](/blogs/android_unity_bridge/47ba7401f40348948acdef43364cf4fa.jpeg)
