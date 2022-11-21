# ElementPlus

## 安装

```npm
npm install element-plus -s
```

## 引入

main.js

```js
import ElementPlus from 'element-plus';
import 'element-plus/dist/index.css'
import zhCn from 'element-plus/es/locale/lang/zh-cn'

createApp(App).use(ElementPlus, {locale: zhCn,})
```

## Icon组件

```npm
npm install @element-plus/icons-vue -s
```

# gin-utils

## 安装

```npm
npm i gin-utils
```

## 引入CSS

main.js

```js
import 'gin-utils/css/global.css'
```

# vue-clipboard3(剪贴板)

## 安装

```npm
npm install vue-clipboard3 --save
```

## 使用

```js
import useClipboard from 'vue-clipboard3'
const { toClipboard } = useClipboard()

const copy = async () => {
      try {
        await toClipboard('Any text you like')
        console.log('Copied to clipboard')
      } catch (e) {
        console.error(e)
      }
    }

```

# axios

## 安装

```npm
npm install axios -s
```

