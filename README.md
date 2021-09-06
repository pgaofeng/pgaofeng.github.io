## 数据统计



| 序号 | 标题                                                         | 描述                  | 分类     |
| ---- | ------------------------------------------------------------ | --------------------- | -------- |
| 1    | [好用的依赖注入框架-Hilt](https://juejin.cn/post/6970580755520946213) | 总结如何使用Hilt      | Jetpack  |
| 2    | [将Room的使用方式塞到脑子里](https://juejin.cn/post/6992875656707211271) | 总结如何使用Room      | Jetpack  |
| 3    | [在Android中使用Bsdiff实现增量更新 ](https://juejin.cn/post/7004809959724548132) | 实现Android的增量更新 | 增量更新 |



## 遇到的问题

1. `2021-08-20：` 在`nova 7(Android 10,EMUI 11.0.0)`上，`paint.setFakeBoldText(true)`不论传的参数是`true`还是`false`，使用`paint.measureText(text)`获取的结果都是一样的，都是未加粗的字体的宽度。
2. `2021-08-30：` 在使用`WebView`加载本地`assets`中的`html`文件时，快速关闭打开`activity`会显示空白。只要在`onDestroy`的时候调用`webView.destroy()`即可。

