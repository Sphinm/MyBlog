目前所写的小程序是基于 wepy 框架，这里记录一下开发过程中遇到的坑。

### 1、在onLaunch时无法获取到微信小程序实例
原生的小程序是可以在 onLaunch 阶段拿到小程序实例的，最近业务接入阿拉丁统计的时候发现无法获取小程序实例，

### 2、两级页面为同一路由时，后者数据覆盖前者
https://github.com/Tencent/wepy/issues/1631

https://github.com/Tencent/wepy/issues/322

### 3、