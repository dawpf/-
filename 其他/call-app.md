# call-app

callapp-lib 是一个 H5 唤起 APP 的解决方案，能够满足大部分唤起客户端的场景，也预留了扩展口，帮你实现一些定制化的功能。

- 安装

  ```
  npm install --save callapp-lib
  ```

- 使用

  ```javascript
  import CallApp from 'callapp-lib';
  
  let scheme = {
  	protocol: 'APP协议,URLScheme的scheme字段,就是你要打开的APP的标识',
  	host: 'URL Scheme的host字段'
  }
  
  const option = {
  	scheme: scheme,
  	intent: { // 安卓原生谷歌浏览器必须传递Intent协议地址,才能唤起APP。
  		package: 'com.xx.xxx.xxxx', // Android包名
  		scheme: 'weixin://', // 和protocol一样:APP协议,URL Scheme的scheme字段,就是你要打开的APP的标识
  	},
  	appstore: 'appstore的下载地址',
  	yingyongbao: '应用宝的下载地址',
  	fallback: '唤端失败后跳转的地址',
  	timeout: 3000,
  };
  
  const lib = new CallApp(option);
  
  lib.open({
      path: '', // 需要打开的页面对应的值,若不需要打开特定页面,path传空字符串''就可以。
  
      param: {}, // 打开APP某个页面时需要接收的参数
  
      // 自定义唤端失败回调函数,传递callback会覆盖callapp-lib库中默认的唤端失败处理逻辑
      callback: function () {
          console.log('唤端失败的处理');
      }
  });
  ```

## options

### scheme

类型: `object`
必填: ✅

用来配置 URL Scheme 所必须的那些 v 字段。

- protocol

  类型: `string`
  必填: ✅

  APP 协议，URL Scheme 的 scheme 字段，就是你要打开的 APP 的标识。

- host

  类型: `string`
  必填: ❎

  URL Scheme 的 host 字段。

- port

  类型: `string` | `number`
  必填: ❎

  URL Scheme 的 port 字段。

### protocol

callapp-lib 2.0.0 版本已移除，原先的 protocol 移入到新增的 scheme 属性中

### outChain

类型: `object`
必填: ❎

外链。我们的 APP 的某些功能可能会集成到另一个 APP 中，为了区分它们的协议，会加上一个中间透明页来分发路由，这层中间页的 URL Scheme 对于我们来说就是外链。当然，这里的外链对 Intent 同样生效。

例：`youku://ykshortvideo?url=xxx`

- protocol (2.0.0 版本由原先的 protocal 修改为 protocol，原先的 protocal 是拼写错误)

  同 URL Scheme 的 scheme 字段，在你的 APP 就和上面的 protocol 属性值相同，在其他 APP 打开就传该 APP 的 scheme 标识。

- path

  参考 URL Scheme 的 path 字段，它代表了该 APP 的具体的某个功页面（功能），这里的 path 就是对应的中间页。

- key

  既然只是中间页，它自然要打开我们真正要打开的页面，所以我们需要把要打开的页面的 URL Scheme 传递过去。就像前端从 URL 的 query 字符串里面取值一样，客户端也是从 URL Scheme 里面来取。至于参数 key 定成什么，大家自己去协商吧，上面的示例中 `url` 也只是一个示例。

### intent

类型: `object`
必填: ❎

安卓原生谷歌浏览器必须传递 Intent 协议地址，才能唤起 APP。

它支持以下五个属性，其中 scheme 和 上面的 protocal 一样，其他四个都是 apk 相关信息，其中 package 和 scheme 必传：

- package
- action
- category
- component
- scheme

### universal

类型: `object`
必填: ❎

如果你们的 ios 工程师没有做相应的配置来让 APP 支持 Universal Link，你可以不用传递， *callap-lib* 将会使用 URL Scheme 来替代它。

- host

  你的 Universal Link 的域名，`apple-app-site-association` 文件就放在这个域名对应的服务器上。

- pathKey

  pathKey 就和前面 Intent 的 key 属性一样，只是这里的 pathKey 是客户端用来提取 path 信息的，以便知道调用的是 APP 的哪个页面。这个值也是需要你和 ios 童鞋协商定下来的。

### appstore

类型: `string`
必填: ✅

APP 的 App Store 地址，例： `https://itunes.apple.com/cn/app/id1383186862`。

### yingyongbao

类型: `string`
必填: ❎

APP 的应用宝地址，例：`'//a.app.qq.com/o/simple.jsp?pkgname=com.youku.shortvideo'`。如果不填写，则安卓微信中会直接跳转 fallback

### timeout

类型: `number`
必填: ❎
默认值: 2000

等待唤端的时间（单位: ms），超时则判断为唤端失败。

### fallback

类型: `string`
必填: ✅

唤端失败后跳转的地址。

### logFunc

类型: `function`
必填: ❎

```
(status: 'pending' | 'failure') => void;
```

埋点入口函数。运营同学可能会希望我们在唤端的时候做埋点，将你的埋点函数传递进来，不管唤端成功与否，它都会被执行。当然，你也可以将这个函数另作他用。

这个回调函数会回执行两次，第一次是触发 open 方法，第二次是唤端失败，它有一个入参 status ，它有两个值 `pending` 和 `failure`，分别代表函数触发及唤端失败。

### buildScheme

类型: `function`
必填: ❎

url scheme 自定义拼接函数，内置的 buildScheme 函数是按照 uri 规范来拼接的，如果你们的 app 对 url scheme 有特殊需求，可以自定义这个函数，此函数有两个入参，`(config, options)`, config 是你调用 open 方法是传入的对象，options 是你初始化 callapp-lib 时传入的对象。

## Method

### open

唤端功能。接收一个对象作为参数，该对象支持以下属性：

- path

  类型: `string` 必填: ✅

  需要打开的页面对应的值，URL Scheme 中的 path 部分，参照 [H5 唤起 APP 指南](https://suanmei.github.io/2018/08/23/h5_call_app/) 一文中的解释。

  只想要直接打开 app ，不需要打开特定页面，path 传空字符串 `''` 就可以。

- param

  类型: `object`
  必填: ❎

  打开 APP 某个页面，它需要接收的参数。

- callback 必填: ❎

  类型: `function`

  自定义唤端失败回调函数。传递 `callback` 会覆盖 *callapp-lib* 库中默认的唤端失败处理逻辑。

### generateScheme

接收一个对象作为参数，该对象包含以下属性：

- path
- param

属性含义和 `open` 方法参数的属性一致。

返回 URL Scheme。如果你觉得 *callapp-lib* 的唤端处理方式不符合你的需求，但你又不想费心费力的自己去拼凑 URL Scheme，可以利用这个方法直接生成。

### generateIntent

生成 Intent 地址，接收参数同 `generateScheme` 方法参数。

### generateUniversalLink

生成 Universal Link，接收参数同 `generateScheme` 方法参数。