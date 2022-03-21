# <center>quasar 学习</center>

## 安装@quasar/cli

```shell
# 需要Node.js>=8.9.0
yarn global add @quasar/cli
#or
npm install -g @quasar/cli
#国内的源
cnpm install -g @quasar/cli
```

## 创建项目

```shell
quasar create <folder_name>
```

创建好之后,可以选择配置`quasar.conf.js`:

- quasar 组件,指令和插件用于网站/app;
- 默认的 quasar 语言包(中文就选择 `lang: 'zh-hans'`);
- 选择 icon 库,可以有多个库,但只能有一个用于 quasar 组件;

```js
extras: ["ionicons-v4", "fontawesome-v5"];
```

- 选择默认 icon 集用于 quasar 组件;

```js
framework: {
  // webfont-based example
  iconSet: "fontawesome-v5";
}
```

- 配置开发服务端口,HTTPS 模式,主机名等;
- 配置使用 css 动画;

```js
// embedding all animations
animations: "all";

// or embedding only specific animations
animations: ["bounceInLeft", "bounceOutRight"];
```

- 启动文件列表(这个决定了执行的顺序)

## 创建 APP

- 首先指定模式:

```shell
# cordova项目
quasar mode add cordova
# capacitor项目
quasar mode add capacitor
```

- 然后指定构建平台:

```shell
#cordova项目
quasar dev -m ios #ios
quasar dev -m android # android
# capacitor项目
quasar dev -m capacitor -T ios # ios
quasar dev -m capacitor -T android # android
```
