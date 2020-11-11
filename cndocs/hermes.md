---
id: hermes
title: Using Hermes
---

<a href="https://hermesengine.dev">
  <img width="300" height="300" style="float: right; margin: -30px 4px 0;" src="https://hermesengine.dev/img/logo.svg"/>
</a>

[Hermes](https://hermesengine.dev) 是一个开源的 JavaScript 引擎，针对在 Android 上运行 React Native 应用进行了优化。 对于许多应用，启用 Hermes 将缩短启动时间、减少内存使用和缩小应用大小。此时，Hermes 就相当于是一个具有 **本地** 特性的 React Native，本文将讲述如何启用它。

首先，确定你使用的 React Native 最低为 0.60.4。


如果现有的应用程序基于 React Native 的早期版本，你需要先去升级它。可以去查看 [升级到新的 React Native 版本](/docs/upgrading) 去执行此操作。尤其要确保对 `android/app/build.gradle` 的更改被应用，细节在 [React Native 升级帮助](https://react-native-community.github.io/upgrade-helper/?from=0.59.0)。升级应用程序之后，在尝试切换到 Hermes 之前，确保应用一切正常。

> ## Windows 使用者注意。
>
> Hermes 的要求在 [Microsoft Visual C++ 2015 Redistributable](https://www.microsoft.com/en-us/download/details.aspx?id=48145)

编辑你的 `android/app/build.gradle` 文件并进行如下修改：

```diff
  project.ext.react = [
      entryFile: "index.js",
-     enableHermes: false  // clean and rebuild if changing
+     enableHermes: true  // clean and rebuild if changing
  ]
```

另外，如果你使用的是 ProGuard，你将需要添加以下规则在 `proguard-rules.pro` :

```
-keep class com.facebook.hermes.unicode.** { *; }
-keep class com.facebook.jni.** { *; }
```

下一步，如果你已经至少构建了一次你的应用程序，请 clean the build：

```sh
$ cd android && ./gradlew clean
```

就这样！现在你应该就可以正常开发部署你的应用程序了：

```sh
$ npx react-native run-android
```

> ## Note about Android App Bundles
>
> Android app bundles are supported from react-native 0.62.0 and up.

## Confirming Hermes is in use

If you've just created a new app from scratch you should see if Hermes is enabled in the welcome view:

![Where to find JS engine status in AwesomeProject](assets/HermesApp.jpg)

A `HermesInternal` global variable will be available in JavaScript that can be used to verify that Hermes is in use:

```jsx
const isHermes = () => global.HermesInternal != null;
```

To see the benefits of Hermes, try making a release build/deployment of your app to compare. For example:

```shell
$ npx react-native run-android --variant release
```

This will compile JavaScript to bytecode during build time which will improve your app's startup speed on device.

## Debugging Hermes using Google Chrome's DevTools

Hermes supports the Chrome debugger by implementing the Chrome inspector protocol. This means Chrome's tools can be used to directly debug JavaScript running on Hermes, on an emulator or device.

Chrome connects to Hermes running on device via Metro, so you'll need to know where Metro is listening. Typically this will be on `localhost:8081`, but this is [configurable](https://facebook.github.io/metro/docs/en/configuration). When running `yarn start` the address is written to stdout on startup.

Once you know where the Metro server is listening, you can connect with Chrome using the following steps:

1. Navigate to `chrome://inspect` in a Chrome browser instance.

2. Use the `Configure...` button to add the Metro server address (typically `localhost:8081` as described above).

![Configure button in Chrome DevTools devices page](assets/HermesDebugChromeConfig.png)

![Dialog for adding Chrome DevTools network targets](assets/HermesDebugChromeMetroAddress.png)

3. You should now see a "Hermes React Native" target with an "inspect" link which can be used to bring up debugger. If you don't see the "inspect" link, make sure the Metro server is running. ![Target inspect link](assets/HermesDebugChromeInspect.png)

4. You can now use the Chrome debug tools. For example, to breakpoint the next time some JavaScript is run, click on the pause button and trigger an action in your app which would cause JavaScript to execute. ![Pause button in debug tools](assets/HermesDebugChromePause.png)
