# CodePush热更新组件详细接入教程

## 什么是codepush
> CodePush是一个微软开发的云服务器。通过它，开发者可以直接在用户的设备上部署手机应用更新。CodePush相当于一个中心仓库，开发者可以推送当前的更新（包括JS/HTML/CSS/IMAGE等）到CoduPush，然后应用将会查询是否有更新。

## 接入流程
* 安装 CodePush CLI
* 注册 CodePush账号
* 在CodePush服务器注册App
* RN代码中集成CodePush
* 原生应用中配置CodePush
* 发布更新的版本

## 项目简书详解地址：[https://www.jianshu.com/p/6a5e00d22723](https://www.jianshu.com/p/6a5e00d22723)

### 1、安装 CodePush CLI
> 安装CodePush指令，直接在终端上输入如下命令即可，**注意：**这个CodePush指令只需要全局安装一次即可，如果第一次安装成功了，那后面就不在需要安装

`$ npm install -g code-push-cli`

![image](http://upload-images.jianshu.io/upload_images/6342050-946c6ff16cb9f5c4..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2、注册 CodePush账号
> 注册CodePush账号也很简单，同样是只需简单的执行下面的命令，同样这个注册操作也是全局只需要注册一次即可

`$ code-push register`

**注意：**当执行完上面的命令后，会自动打开一个授权网页，让你选择使用哪种方式进行授权登录，这里我们统一就选择使用GitHub即可

![image](http://upload-images.jianshu.io/upload_images/6342050-2a4306480898b0a7..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当注册成功后，CodePush会给我们一个key

![image](http://upload-images.jianshu.io/upload_images/6342050-750526f901c9f589..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们直接复制这个key，然后在终端中将这个key填写进去即可，填写key登录成功显示效果如下

![image](http://upload-images.jianshu.io/upload_images/6342050-ac9452f71702dbb3..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们使用下面的命令来验证我的登录是否成功

`$ code-push login`

![image](http://upload-images.jianshu.io/upload_images/6342050-e5a88001ca03d4f4..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**CodePush注册登录相关命令：**

* code-push login 登陆
* code-push loout 注销
* code-push access-key ls 列出登陆的token
* code-push access-key rm <accessKye> 删除某个 access-key

### 3、在CodePush服务器注册App
> 为了让CodePush服务器有我们的App，我们需要CodePush注册App，输入下面命令即可完成注册，这里需要注意如果我们的应用分为iOS和Android两个平台，这时我们需要分别注册两套key
> 应用添加成功后就会返回对应的`production` 和 `Staging` 两个key，`production`代表生产版的热更新部署，`Staging`代表开发版的热更新部署，在ios中将staging的部署key复制在info.plist的CodePushDeploymentKey值中，在android中复制在Application的getPackages的CodePush中

*添加iOS平台应用*

`$ code-push app add iOSRNHybrid ios react-native`

![image](http://upload-images.jianshu.io/upload_images/6342050-68a3f5000342815b..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*添加Android平台应用*

```
$ code-push app add iOSRNHybridForAndroid Android react-native
```

![image](http://upload-images.jianshu.io/upload_images/6342050-27693de751a1b047..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以输入如下命令来查看我们刚刚添加的App

`$ code-push app list`

![image](http://upload-images.jianshu.io/upload_images/6342050-844ee8a8db1edb36..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**CodePush管理App的相关命令：**

* code-push app add 在账号里面添加一个新的app
* code-push app remove 或者 rm 在账号里移除一个app
* code-push app rename 重命名一个存在app
* code-push app list 或则 ls 列出账号下面的所有app
* code-push app transfer 把app的所有权转移到另外一个账号

### 4、RN代码中集成CodePush
> 首先我们需要安装CodeoPush组件，然后通过link命令添加原生依赖，最后在RN根组件中添加热更新逻辑代码

*安装组件*

`$ npm install react-native-code-push --save `

![image](http://upload-images.jianshu.io/upload_images/6342050-293f7a44ed3e1444..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*添加原生依赖，这里添加依赖我们使用自动添加依赖的方式*

`$ npm link react-native-code-push`

![image](http://upload-images.jianshu.io/upload_images/6342050-94215effebb35381..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*我们在RN项目的根组件中添加热更新逻辑代码如下*

```
import React, { Component } from 'react';
import {
  Platform,
  StyleSheet,
  Text,
  View
} from 'react-native';

import CodePush from "react-native-code-push"; // 引入code-push

let codePushOptions = {
  //设置检查更新的频率
  //ON_APP_RESUME APP恢复到前台的时候
  //ON_APP_START APP开启的时候
  //MANUAL 手动检查
  checkFrequency : CodePush.CheckFrequency.ON_APP_RESUME
};

const instructions = Platform.select({
  ios: 'Press Cmd+R to reload,\n' +
    'Cmd+D or shake for dev menu',
  android: 'Double tap R on your keyboard to reload,\n' +
    'Shake or press menu button for dev menu',
});

type Props = {};

class App extends Component<Props> {

  //如果有更新的提示
  syncImmediate() {
    CodePush.sync( {
          //安装模式
          //ON_NEXT_RESUME 下次恢复到前台时
          //ON_NEXT_RESTART 下一次重启时
          //IMMEDIATE 马上更新
          installMode : CodePush.InstallMode.IMMEDIATE ,
          //对话框
          updateDialog : {
            //是否显示更新描述
            appendReleaseDescription : true ,
            //更新描述的前缀。 默认为"Description"
            descriptionPrefix : "更新内容：" ,
            //强制更新按钮文字，默认为continue
            mandatoryContinueButtonLabel : "立即更新" ,
            //强制更新时的信息. 默认为"An update is available that must be installed."
            mandatoryUpdateMessage : "必须更新后才能使用" ,
            //非强制更新时，按钮文字,默认为"ignore"
            optionalIgnoreButtonLabel : '稍后' ,
            //非强制更新时，确认按钮文字. 默认为"Install"
            optionalInstallButtonLabel : '后台更新' ,
            //非强制更新时，检查到更新的消息文本
            optionalUpdateMessage : '有新版本了，是否更新？' ,
            //Alert窗口的标题
            title : '更新提示'
          } ,
        } ,
    );
  }

  componentWillMount() {
    CodePush.disallowRestart();//页禁止重启
    this.syncImmediate(); //开始检查更新
  }

  componentDidMount() {
    CodePush.allowRestart();//在加载完了，允许重启
  }

  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>
          Welcome to React Native!
        </Text>
        <Text style={styles.instructions}>
          To get started, edit App.js
        </Text>
        <Text style={styles.instructions}>
          {instructions}
        </Text>

        <Text style={styles.instructions}>
          这是更新的版本
        </Text>
      </View>
    );
  }
}

// 这一行必须要写
App = CodePush(codePushOptions)(App)

export default App

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },
})
```

### 5、原生应用中配置CodePush
> 这里原生应用中配置CodePush我们需要分别配置iOS平台和Android平台

#### 配置iOS平台
* 使用Xcode打开项目，Xcode的项目导航视图中的PROJECT下选择你的项目，选择Info页签 ，在Configurations节点下单击 + 按钮 ，选择Duplicate "Release Configaration，输入Staging

![image](http://upload-images.jianshu.io/upload_images/6342050-fbfc9b79199d4a3f..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 选择Build Settings tab，搜索Build Location -> Per-configuration Build Products Path -> Staging，将之前的值：`$(BUILD_DIR)/$(CONFIGURATION)$(EFFECTIVE_PLATFORM_NAME)` 改为：`$(BUILD_DIR)/Release$(EFFECTIVE_PLATFORM_NAME)`

![image](http://upload-images.jianshu.io/upload_images/6342050-369c31244b0f528f..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 选择Build Settings tab，点击 `+` 号，选择`Add User-Defined Setting`，将key设置为`CODEPUSH_KEY`，Release 和 Staging的值为前面创建的key，我们直接复制进去即可

![image](http://upload-images.jianshu.io/upload_images/6342050-663dc2df47438cc5..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 打开Info.plist文件，在CodePushDeploymentKey中输入`$(CODEPUSH_KEY)`，并修改Bundle versions为三位

![image](http://upload-images.jianshu.io/upload_images/6342050-c4392091edc647a0..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

iOS平台CodePush环境集成完毕

#### 配置Android平台

### 6、发布更新的版本
> 在使用之前需要考虑的是检查更新时机，更新是否强制，更新是否要求即时等

#### 更新时机
>  一般常见的应用内更新时机分为两种，一种是打开App就检查更新，一种是放在设置界面让用户主动检查更新并安装

* 打开APP就检查更新

	最为简单的使用方式在React Natvie的根组件的componentDidMount方法中通过
	codePush.sync()（需要先导入codePush包：import codePush from 'react-native-code-push'）方法检查并安装更新，如果有更新包可供下载则会在重启后生效。不过这种下载和安装都是静默的，即用户不可见。如果需要用户可见则需要额外的配置。具体可以参考codePush官方API文档，部分代码，完整代码请参照文档上面
	
	```
	codePush.sync({
      updateDialog: {
        appendReleaseDescription: true,
        descriptionPrefix:'\n\n更新内容：\n',
        title:'更新',
        mandatoryUpdateMessage:'',
        mandatoryContinueButtonLabel:'更新',
      },
      mandatoryInstallMode:codePush.InstallMode.IMMEDIATE,
      deploymentKey: CODE_PUSH_PRODUCTION_KEY,
    });
	```
	
	上面的配置在检查更新时会弹出提示对话框，  mandatoryInstallMode表示强制更新，appendReleaseDescription表示在发布更新时的描述会显示到更新对话框上让用户可见
	
* 用户点击检查更新按钮
	
	在用户点击检查更新按钮后进行检查，如果有更新则弹出提示框让用户选择是否更新，如果用户点击立即更新按钮，则会进行安装包的下载（实际上这时候应该显示下载进度，这里省略了）下载完成后会立即重启并生效（也可配置稍后重启），部分代码如下
	
	```
	codePush.checkForUpdate(deploymentKey).then((update) => {
        if (!update) {
            Alert.alert("提示", "已是最新版本--", [
                {
                    text: "Ok", onPress: () => {
                    console.log("点了OK");
                }
                }
            ]);
        } else {
            codePush.sync({
                    deploymentKey: deploymentKey,
                    updateDialog: {
                        optionalIgnoreButtonLabel: '稍后',
                        optionalInstallButtonLabel: '立即更新',
                        optionalUpdateMessage: '有新版本了，是否更新？',
                        title: '更新提示'
                    },
                    installMode: codePush.InstallMode.IMMEDIATE,

                },
                (status) => {
                    switch (status) {
                        case codePush.SyncStatus.DOWNLOADING_PACKAGE:
                            console.log("DOWNLOADING_PACKAGE");
                            break;
                        case codePush.SyncStatus.INSTALLING_UPDATE:
                            console.log(" INSTALLING_UPDATE");
                            break;
                    }
                },
                (progress) => {
                    console.log(progress.receivedBytes + " of " + progress.totalBytes + " received.");
                }
            );
        }
     }
	```
	
#### 更新是否强制
> 如果是强制更新需要在发布的时候指定，发布命令中配置--m true

#### 更新是否要求即时
> 在更新配置中通过指定installMode来决定安装完成的重启时机，亦即更新生效时机

* codePush.InstallMode.IMMEDIATE ：安装完成立即重启更新
* codePush.InstallMode.ON_NEXT_RESTART ：安装完成后会在下次重启后进行更新
* codePush.InstallMode.ON_NEXT_RESUME ：安装完成后会在应用进入后台后重启更新

#### 如何发布CodePush更新包
> 在将RN的bundle放到CodePush服务器之前，我们需要先生成bundle，在将bundle上传到CodePush

**生成bundle**

* 我们在RN项目根目录下线创建bundle文件夹，再在bundle中创建创建ios和android文件夹，最后将生成的bundle文件和资源文件拖到我们的项目工程中

![image](http://upload-images.jianshu.io/upload_images/6342050-5a4f6eec95bd4879..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 生成bundle命令 `react-native bundle --platform 平台 --entry-file 启动文件 --bundle-output 打包js输出文件 --assets-dest 资源输出目录 --dev 是否调试`

```
$ react-native bundle --entry-file index.ios.js --bundle-output ./bundle/ios/main.jsbundle --platform ios --assets-dest ./bundle/ios --dev false
```

![image](http://upload-images.jianshu.io/upload_images/6342050-05e5bd76d4950e84..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 将生成的bundle文件和资源文件拖到我们的项目工程

![image](http://upload-images.jianshu.io/upload_images/6342050-a53124c0033838b9..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**上传bundle**

* 将生成的bundle文件上传到CodePush，我们直接执行下面的命令即可

`$ code-push release-react <Appname> <Platform> --t <本更新包面向的旧版本号> --des <本次更新说明>`

**注意：** CodePush默认是更新Staging 环境的，如果发布生产环境的更新包，需要指定--d参数：--d Production，如果发布的是强制更新包，需要加上 --m true强制更新

```
$ code-push release-react iOSRNHybrid ios --t 1.0.0 --dev false --d Production --des "这是第一个更新包" --m true
```

更新包上传到CodePush服务器成功后，效果图如下：
![image](http://upload-images.jianshu.io/upload_images/6342050-f2efb3f1466eea8e..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看发布的历史记录，命令如下

`$ code-push deployment history ProjName Staging`

![image](http://upload-images.jianshu.io/upload_images/6342050-ddc6a0a89b3eda27..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**对1.0.0版本的应用如何发布第二个、第n个更新包**
> 操作步骤和上面发布第一个更新包流程一样，我们任然先需要打出bundle包，然后再将bundle发布到CodePush

```
$ react-native bundle --entry-file index.ios.js --bundle-output ./bundle/ios/main.jsbundle --platform ios --assets-dest ./bundle/ios --dev false
```

```
$ code-push release-react iOSRNHybrid ios --t 1.0.0 --dev false --d Production --des "这是第二个更新包" --m true
```

## 注意事项
* 当我们在生成更新包之前，我们需要先将JS代码打包成bundle，然后拖拽到项目中，打包之前我们需要先自己建立输出bundle的文件夹`bundle -> ios`，打bundle命令如下：

```
$ react-native bundle --entry-file index.ios.js --bundle-output ./bundle/ios/main.jsbundle --platform ios --assets-dest ./bundle/ios --dev false
```

![image](http://upload-images.jianshu.io/upload_images/6342050-f06b0d746cf909cc..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 发布更新包命令中的 `-- t` 对应的参数是和我们项目中的版本号一致的，这个不要误理解为是更新包的版本号，例如项目中的版本号为`1.0.0`， 这时如果我们需要对这个`1.0.0` 版本的项目进行第一次热更新，那么命令中的 `-- t` 也为`1.0.0`，第二次热更新任然为`1.0.0`

* 项目的版本号需要改为三位的，默认是两位的，但是CodePush需要三位数的版本号

* 发布更新应用时，应用的名称必须要和之前注册过的应用名称一致

![image](http://upload-images.jianshu.io/upload_images/6342050-4a1b833d90a961f0..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 创建应用时，信息要填写正确

![image](http://upload-images.jianshu.io/upload_images/6342050-33a61abfa0c2398b..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 当执行link，命令卡住不执行时，这时直接按回车键先ignore key即可

![image](http://upload-images.jianshu.io/upload_images/6342050-c62e758da8a48814..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 福利时间
* 作者React Native开源项目OneM地址(按照企业开发标准搭建框架完成开发的)：**[https://github.com/guangqiang-liu/OneM](https://github.com/guangqiang-liu/OneM)**：欢迎小伙伴们 **star**
* 作者简书主页：包含50多篇RN开发相关的技术文章[http://www.jianshu.com/u/023338566ca5](http://www.jianshu.com/u/023338566ca5) 欢迎小伙伴们：**多多关注**，**多多点赞**
* 作者React Native QQ技术交流群：**620792950** 欢迎小伙伴进群交流学习
* 友情提示：**在开发中有遇到RN相关的技术问题，欢迎小伙伴加入交流群（**620792950**），在群里提问、互相交流学习。交流群也定期更新最新的RN学习资料给大家，谢谢大家支持！**