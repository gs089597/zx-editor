# ZxEditor

移动端HTML文档（富文本）编辑器，支持图文混排、引用、大标题、无序列表，字体颜色、加粗、斜体。

可用于独立web项目开发，也可以用于与原生App混合(hybrid)开发。

![ZxEditor](resource/preview2.png)

![ZxEditor](resource/preview1.jpg)

## 本地运行 Build Setup

``` bash
# 安装依赖
npm i

# 运行开发模式
# 通过 `http://localhost:9001` 可以访问
npm run dev

# 生成项目文件
npm run build
```

## 使用 Use

* npm

```bash
  npm install zx-editor --save-dev
```

* html

```html
<div id="editorContainer">
  <!-- 编辑器容器 -->
</div>
<script src="./dist/js/zx-editor.min.js"></script>
<script>
  // 初始化ZX编辑器
  var zxEditor = new ZxEditor('#editorContainer', {
    fixed: true
  })
  // 详见index.html文件
</script>
```

注意：添加照片时，判断照片方向，并自动旋转需要依赖插件 exif.js

* ES6+

```javascript
import { ZxEditor } from 'zx-editor'
// import { ZxEditor } from './src/js/zx-editor/index.js'
```

## 参数 options

* autoSave: `Number`，自动保存编辑内容至localStorage，单位秒。等于0则不自动保存编辑内容。

* fixed: `Boolean`，编辑器是否绝对定位，默认为false。

* top: `Number`，顶部距离，绝对定位时，相对于WebView顶部的距离。

* bottom: `Number`，底部距离。

* padding: `Number`，编辑器左右内边距，默认15像素,

* showToolbar: `Boolean`，是否显示底部工具栏（图片、标签、连接添加等图标）。默认为`true`

## 方法 methods

* on(notifyName, callback)

  监听编辑器部分通知

  |notifyName|callback(参数)|说明|
  |:--|:--|:--|
  |add-link|`next()`|点击底部`添加连接`图标时触发。监听此通知，将阻止编辑器默认处理逻辑执行|
  |debug|messge|消息通知|
  |error|errorMessage|错误异常通知|
  |select-picture|-|点击底部`选择图片`图标时触发。监听此通知，将阻止编辑器默认处理逻辑执行|
  |show-emoji|-|点击底部`emoji`图标时触发|
  |show-textstyle|-|点击底部`文字样式`图标时触发|

  例子：

  ```
  // 自定义选择图片
  zxEditor.on('select-picture', _ => {
    // hybrid模式开发时，此处可以调用原生App提供的接口，访问图片文件选择列表
    // 执行图片文件数据获取，
    // 或者获取由原生App处理并上传完成的图片url
    zxEditor.addImage('图片url地址或base64图片数据')
    // 其他操作...
  })
  ```

  ```
  // 自定义添加链接
  zxEditor.on('add-link', next => {
    // hybrid模式开发时，此处可以调用原生App提供的接口，访问剪贴板是否有url地址数据
    // 获取到url地址、及其文档title
    // 将链接添加至编辑器中
    next(url, title)
  })
  ```

* addFooterButton(option)

  底部工具栏添加按钮，`option`: `Object|Array`

  ```
  // 底部工具栏添加一个“导语”按钮
  zxEditor.addFooterButton({
    title: '导语',
    // 按钮外容器样式
    class: 'demo-summary-button',
    // 按钮内i元素样式名
    icon: '',
    // 需要注册的监听事件名
    on: 'summary-button'
  })

  // 或者
  zxEditor.addFooterButton(
    [
      {
        title: '导语',
        // 按钮外容器样式
        class: 'demo-summary-button',
        // 按钮内i元素样式名
        icon: '',
        // 需要注册的监听事件名
        on: 'summary-button'
      }
    ]
  )
  ```

* addImage(src|base64)

  向正文焦点处添加一张图片，支持图片url地址或base64数据

* getBase64Images()

  获取正文中所有base64数据的图片，返回一个数组

  @return array

  ```
    [
      {
        id: 'zxEditor_img_1500001511111',
        base64: 'data:image/jpeg;base64,/9j4AAQSkZJDAAkGB+wgH....',
        blob: 'Blob数据，可以用于直接上传，或通过方法toBlobData(base64)转换'
      }
    ]
  ```

* toBlobData(base64)

  将图片base64转换为原始数据类型Blob()，该数据和表单中提交上传的数据同类型，故可以直接上传

  @return new Blob() 返回Blob()数据

* setImageSrc(imgId, imgUrl)

  将ID为imgId的图片base64地址，替换为新的imgUrl。需配合`getBase64Images()`方法使用。

  @return boolean

* getContent(isInnerText)

  获取正文内容html。

  @params 'isInnerText'可选，默认为`false`，获取编辑器innerHTML。否则获取innerText。

* setContent(innerHTML)

  设置编辑器内容，可用于初始化编辑器数据。

* stopAutoSave()

  开启自动保存时有效。停止自动保存编辑数据。

* save()

  保存编辑器内容

* removeSave()

  移除本地存储的content内容

* filesToBase64(files, opts, callback)

  图片文件数据转为base64/blob

  |参数|类型|说明|
  |:--|:--|:--|
  |files|文件类型|文件数据数组|
  |opts|`Object`|图片压缩或裁剪参数`{width:100,height:100,clip:true}`|
  |callback(errorArray, dataArray)|errorArray:`null|Array`, dataArray:`null|Array`|文件处理完成回调函数|

  dataArray

  ```
  null
  // 或者
  [
    {
      // 处理完成的dom节点对象
      element: '$canvas',
      // 文件类型
      type: 'image/png',
      // 处理完成的图片宽度，根据传入的参数定
      width: 100,
      // 处理完成的图片高度
      height: 100,
      // blob数据
      data: 'blob数据',
      // base64数据
      base64: 'base64',
      // 文件大小B
      size: 15455,
      // blob url地址
      url: 'blobUrl',
      // 原始图片数据参数
      rawdata: {}
    }
  ]
  ```


## Copyright and license

Code and documentation copyright 2018. zx1984. Code released under the MIT License.



