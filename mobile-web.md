# 四种移动开发模式

1. Mobile Web: 以Html5, Css3, JavaScript为基础，依托浏览器作为宿主运行环境的应用程序，Html5开发
2. Hybrid Web: 本质也是Mobile Web页面开发，它以webview渲染用户界面，通过JSBridge等中间件的方式跟底层客户端API进行双向通行，使得应用具备了客户端的操作能力
3. Reactive Native: 近两年发展的开发框架，类似框架有weex, NativeScript
4. Native: 基于诸如Java或者Object-C编程语言的直接使用手机系统API进行开发


## ionic部署到特定的设备上

1. ionic cordova run ios --device
2. ionic cordova run ios --list
3. ionic cordova run ios --release --buildConfig=myBuildConfig.json --target=iPhone-5

## ionic 模拟

1. ionic cordova emulate
2. ionic cordova emulate ios
3. ionic cordova emulate android

## ionic plugin management

1. ionic cordova plugin ls
2. ionic cordova plugin add cordova-plugin-camer@^2.0.0
3. ionic cordova plugin add https://github.com/myfork/cordova-plugin-camera.git#2.1.0
4. ionic cordova plugin add ../cordova-plugin-camera
5. ionic cordova plugin add ../cordova-plugin-camera.tgz
6. ionic cordova plugin rm camera


## ionic 打包

1. ionic cordova resources
2. ionic cordova resources android
3. ionic cordova resources -i

## platform 支持

1. ionic cordova platform ls
2. ionic cordova platform add android
3. ionic cordova platform add ios


## ionic 生成

1. ionic cordova build android --debug --device
2. ionic cordova build android --release --buildConfig=myBuildConfig.json
3. ionic cordova build andorid --release -- --keystore="andorid.keystore" --storePassword=android --aliasmykey

