## vue-ssr-template

> 用于开发时快速创建 [**vue-ssr**](https://ssr.vuejs.org/zh/) 项目开发脚手架

### Node 版本

```
v10.15.3
```

<br/>

---

## 从零搭建Vue-SSR项目
<br/>


#### 一、首先新建一个文件夹 vue-ssr，然后初始化项目
```	bash
npm init -y
```
<br/>


#### 二、然后先新建一个 src 文件夹，里面新建一个 `app.js` 入口文件和 App.vue 最高组件，和 components 文件夹

```
-- vue-ssr

   -- src 文件夹
	
   -- components 文件夹
	
      app.js

      App.vue
```
<br/>



#### 三、ssr  所需要的模块

**生产环境模块：**	
> [***vue***](https://github.com/vuejs/vue)、[***vue-server-renderer***](https://www.npmjs.com/package/vue-server-renderer)、[***vue-router***](https://github.com/vuejs/vue-router)、[***express***](https://github.com/expressjs/express)

**开发环境模块：**
> [***babel-core***](https://www.npmjs.com/package/babel-core)、[***babel-loader***](https://github.com/babel/babel-loader)、[***babel-preset-env***](https://github.com/babel/babel-preset-env)

> [***babel-plugin-syntax-dynamic-import***](https://www.npmjs.com/package/babel-plugin-syntax-dynamic-import)

> [***css-loader***](https://www.npmjs.com/package/css-loader)、[***vue-style-loader***](https://www.npmjs.com/package/vue-style-loader)、[***sass-loader***](https://www.npmjs.com/package/sass-loader)、[***node-sass***](https://www.npmjs.com/package/node-sass)

> [***vue-loader***](https://www.npmjs.com/package/vue-loader)、[***vue-template-loader***](https://www.npmjs.com/package/vue-template-loader)、[***vue-template-compiler***](https://www.npmjs.com/package/vue-template-compiler)

> [***webpack***](https://www.webpackjs.com/)、[***webpack-cli***](https://www.webpackjs.com/)、[***webpack-merge***](https://www.webpackjs.com/)

<br/>

> 以下模块版本可以作为参考

```javascript
// package.json

{
  "name": "vue-ssr-template",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
      ....
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.16.3",
    "vue": "^2.6.10",
    "vue-router": "^3.0.1",
    "vue-server-renderer": "^2.6.10",
    "vuex": "^3.1.0",
    "vuex-router-sync": "^5.0.0"
  },
  "devDependencies": {
    "babel-core": "^6.26.3",
    "babel-loader": "^7.1.5",
    "babel-plugin-syntax-dynamic-import": "^6.18.0",
    "babel-preset-env": "^1.7.0",
    "css-loader": "^1.0.0",
    "node-sass": "^4.12.0",
    "sass-loader": "^7.0.3",
    "vue-loader": "^15.2.4",
    "vue-style-loader": "^4.1.0",
    "vue-template-compiler": "^2.6.10",
    "vue-template-loader": "^1.0.0",
    "webpack": "^4.30.0",
    "webpack-cli": "^3.0.8",
    "webpack-merge": "^4.1.3"
  }
}
```

<br/>


#### 四、先编写服务端渲染

> **注意**

> 因为 **ssr** 服务端渲染需要有服务端代码和客户端代码混合一起使用，所以需要公用同一个 `app.js `入口文件

<br/>

**1、** 先安装 ***vue***、***vue-server-renderer***、***vue-router***、***express*** 模块

```bash
npm install vue vue-server-renderer vue-router express --save
```
<br/>

**2.** 然后编写 `app.js` 公共入口文件代码，这是一个公共入口，可供客户端和服务端，里面是 **Vue** 实例

```javascript
import Vue from "vue"
import App from "./App.vue"

// 防止多个请求之间共享一个实例，容易导致交叉污染
// 需要暴露一个可重复执行的工厂函数，为每个请求创建新的应用程序实例

export function createApp () {

    const app = new Vue({
        render: h => h(App)
    })

    return { app }
}
```
<br/>


**3.** 然后把 **App.vue** 文件编写一下
```html
<template>
    <div class="app">
        <h1>欢迎来到VUE-SSR</h1>
    </div>
</template>
<script>
export default {
    name: "App"
}
</script>
<style>
</style>
```
<br/>


**4.** 然后在根目录添加一个 **entry** 文件夹，用来存放客户端和服务端的 **webpack** 打包入口文件

> 客户端入口：`entry-client.js`

> 服务端入口：`entry-server.js`

<br/>

**5.** 编写 `entry-server.js` 文件
```javascript
// 先引入 app.js 公共入口文件
import { createApp } from "../src/app.js"


// 然后暴露一个唯一的匿名函数
export default () => {

    return new Promise((resolve, reject) => {

        const { app } = createApp()

        // 返回一个 Vue 的实例
        resolve(app)
    })
}
```

<br/>

**6.** 然后在根目录新建一个 **build** 文件夹，里面存放 **webpack** 对服务端和客户端打包的配置

> 客户端打包配置：`webpack.client.js`

> 服务端打包配置：`webpack.server.js`

> 公共打包配置：`webpack.common.js`

<br/>

**7.** 先编写 `webpack.common.js` 公共配置

> **7.1.** 先安装 ***webpack*** 、***vue-loader*** 、***babel*** 、***sass*** 的模块

```bash
npm install webpack webpack-cli webpack-merge --save-dev

            vue-loader vue-template-loader vue-template-compiler

            sass-loader node-sass vue-style-loader css-loader

            babel-core babel-loader babel-preset-env
```

> **注意**

> 下载了下面这个模块才可以使用路由按需加载，如果 **npm** 下载不了的话，可以使用 **cnpm** 下载

```bash
cnpm install @babel/plugin-syntax-dynamic-import --save-dev
```
<br/>

**7.2.** 然后编写公共配置代码

> **注意**

> 渲染 **css** 代码一定要使用 ***vue-style-loader*** 模块，不然会报错

```javascript
// webpack.common.js

const path = require("path")
const VueLoaderPlugin = require("vue-loader/lib/plugin")


module.exports = {
    entry: "",
    output: {
	    path: path.resolve(__dirname, "../dist/")
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: "babel-loader",
                exclude: /node_modules/,
                options: {
                    presets: ["env"],

                    // 使用vue-router按需加载语法
                    plugins: ["syntax-dynamic-import"]
                }
            },
            {
                test: /\.css$/,

                // 在后面加?sourceMap是为了调试css
                loader: "vue-style-loader?sourceMap!css-loader?sourceMap"
            },
            {
                test: /\.scss$/,

                // 在后面加?sourceMap是为了调试css
                loader: "vue-style-loader?sourceMap!css-loader?sourceMap!sass-loader?sourceMap"
            },
            {
                test: /\.vue$/,
                loader: "vue-loader"
            }
        ]
    },
    resolve: {
        alias: {
            "vue$": "vue/dist/vue.esm.js"
        }
    },
    plugins: [
        new VueLoaderPlugin()
    ]
}
```
<br/>

**8.** 然后配置 `webpack.server.js` 服务端代码打包配置

```javascript
const merge = require("webpack-merge")
const common = require("./webpack.common.js")
const path = require("path")


module.exports = merge(common, {

    // 此处告知 webpack 打包的代码是运行在 node 端的
    target: "node",

    entry: path.resolve(__dirname, "../entry/entry-server.js"),
    output: {
        filename: "bundle-server.js",
        libraryTarget: "commonjs2"
    },

    // 设置打包时忽略 package.json 里的 dependencies 里的模块
    externals: Object.keys(require("../package.json").dependencies)
})
```
<br/>

**9.** 现在开始编 **node** 端代码

> **9.1.** 先安装服务端渲染需要的模块

```bash
npm install vue-server-renderer express --save
```
<br/>

> **9.2.** 然后在根目录新建一个 `server.js` 文件，编写代码

```javascript
const express = require("express")
const server = express()

// 使用打包后的代码来渲染
const createApp = require("./dist/bundle-server")["default"]

// 关键：需要此模块把 vue 实例转化成 html 代码
const renderer = require("vue-server-renderer").createRenderer()


server.get("*", (req, res) => {
    
    // 创建 vue 实例，传入请求路由信息
    createApp().then((app) => {
        renderer.renderToString(app, (err, html) => {
            if (err) {
                console.log(err)
                return res.send(`运行时错误${err}`)
            }
            console.log(html)
            res.send(`
                <!DOCTYOE html>
                <html>
                    <head>
                        <meta chart="utf8" />
                        <title>VUE-SSR</title>
                    </head>
                    <body>
                        <div id="app">${html}</div>
                    </body>
                </html>
            `)
        })
    })
})

server.listen(8080, () => {
    console.log("服务端渲染端口号为：8080")
})
```
<br/>			
		
			
#### 五、现在可以在 package.json 的 scripts 里添加代码

> **1.** **package.json** 的 `scripts` 里添加代码
```json
"scripts": {
    "build-server": "webpack --config build/webpack.server.js",
    "start": "node server.js"
}
```

> **2.** 然后可以在终端里输入代码，运行项目了

```bash
npm run build-server

npm start
```
<br/>


#### 六、现在把 vue-router 加入到服务端渲染代码里


> **1.** 先修改 **App.vue** 文件代码

```html
· App.vue

<template>
    <div class="app">
        <h1>欢迎来到VUE-SSR</h1>

        // 为后面设置的路由做好准备
        <router-link to="/">首页</router-link>
        <router-link to="/container">内容</router-link>

        <router-view></router-view>
    </div>
</template>
<script>
export default {
    name: "App"
}
</script>
<style>
</style>
```
<br/>

> **2.** 然后在 **src** 目录里的 **components** 文件夹里新建 **Home.vue** 和 **Container.vue** 文件，编写代码
			
```html
· Home.vue

<template>
    <div class="home">
        <h1>Home</h1>
    </div>
</template>
<script>
export default {
    name: "Home"
}
</script>
<style>
</style>
```
```html
· Container.vue

<template>
    <div class="home">
        <h1>Home</h1>
    </div>
</template>
<script>
export default {
    name: "Home"
}
</script>
<style>
</style>
```
<br/>

> **2.** 然后在 **src** 目录里新建一个 `router.js` 文件，编写代码

```javascript
import Vue from "vue"
import VueRouter from "vue-router"

Vue.use(VueRouter)

export function createRouter () {
    return new VueRouter({
        mode: "history",
        routes: [
            {
                path: "/",
                component: () => import("./components/Home.vue")
            },
            {
                path: "/container",
                component: () => import("./components/Container.vue")
            }
        ]
    })
}
```
<br/>

> **3.** 然后回到 `app.js` 里修改代码

> .... （此为省略代码）

```javascript
.... (此为省略代码)
		
import { createRouter } from "./router.js"

export function createApp () {	    
    const router = createRouter()
    ....

    const app = new Vue({
        router,
        render: h => h(App)
    })

    return { app, router }
}
```	
<br/>
	
> **4.** 在回到 `entry-server.js` 里修改代码

```javascript
const { createApp } = require("../src/app.js")

export default (context) => {

    return new Promise((resolve, reject) => {
        const { app, router } = createApp()

        // 更换路由
        router.push(context.url)


        // 等到 router 将可能的异步组件和钩子函数解析完
        router.onReady(() => {

            // 获取相应的路由下的组件
            const matchedComponents = router.getMatchedComponents()

            // 如果没有组件，说明该路由不存在，报错404
            if (!matchedComponents.length) {
                return reject("没有此页面")
            }

            resolve(app)

        }, reject)
    })
}
```
<br/>
	
> **5.** 然后到 `server.js` 文件里修改代码

```javascript
.... (省略代码)

server.get("*", (req, res) => {

    // 把请求的路由传到服务端渲染的入口文件里判断是否存在此路由
    const context = {
        url: req.url
    }
    
    // 通过传参把请求的路由传过去判断
    createApp(context).then((app) => {
        ....
    })
})
```


> **6.** 现在就可以运行项目了

```bash
npm run build-server

npm start
```
<br/>


#### 七、现在把服务端渲染做了一半，现在需要做客户端渲染


> **1.** 到 **entry** 文件夹里，编写 `entry-client.js` 文件

```javascript
import { createApp } from "../src/app.js"

const { app } = createApp()

// 等到 router 将可能的异步组件和钩子函数解析完
router.onReady(() => {
    app.$mount("#app")
})
```
<br/>

> **2.** 编写根目录的 **build** 文件夹里的 `webpack.client.js` 文件

```javascript
const merge = require("webpack-merge")
const common = require("./webpack.common.js")
const path = require("path")

module.exports = merge(common, {
    entry: path.resolve(__dirname, "../entry/entry-client.js"),
    output: {
        filename: "bundle-client.js"
    }
})
```

> **3.** 然后再去根目录的 `package.json` 里面添加命令

```json
"scripts": {
    ....

    "build-client": "webpack --config build/webpack.client.js"
}
```

> **4.** 客户端激活，再到根目录的 `server.js` 文件里添加代码

```javascript
· 把打包的 js 代码放到 head ，因为 css 代码也会打包进 js 文件里

· 所以为了避免页面渲染一开始会出现没有样式的页面，从而把 js 放在头部加载

....(省略代码)

// 设置静态资源目录
server.use("/dist", express.static(__dirname + "/dist"))
const clientBundleFileUrl = "/dist/bundle-client.js"


server.get("*", (req, res) => {
    ....

    createApp(context).then((app) => {
        renderer.renderToString(app, (err, html) => {
        ....

            res.send(`
                <!DOCTYPE html>
                <html>
                    <head>
                        <meta chart="utf8" />
                        <title>VUE-SSR</title>

                        // 客户端激活
                        <script src="${clientBundleFileUrl}"></script>
                    </head>
                    <body>
                        <div id="app">${html}</div>
                    </body>
                </html>
            `)
        })
    }).catch((err) => {
        console.log(`路由404错误：${err}`)
        res.send(`路由404错误：${err}`)
    })
})
```
<br/>

> **5.** 然后就可以运行项目查看效果了

```bash
npm run build-client

npm run build-server

npm start
```

> 可以发现页面只有一开始加载的时候会从服务器加载 html 代码，切换路由都不会请求服务器

> 这就是客户端激活后的效果

<br/>


#### 八、如果在服务端项目想要引入 bootstrap、jquery 等第三方库的话，需要在 entry-client.js 文件里import

> **注意**： 这是使用 **npm** 下载的方式

> 引入 `bootstrap.min.js` 还需要下载 `popper.js`、`jquery` 模块

```javascript
npm install popper.js --save-dev

npm install jquery --save


// entry-client.js
....

import "../src/assets/bootstrap/bootstrap.min.js"
import "../src/assets/bootstrap/bootstrap.min.css"

....

* 然后才可以打包，运行服务端渲染
```
<br/>

#### 九、使用 html 模板

```html
· index.template.html

<!DOCTYPE html>
<html lang="en">
   <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>{{title}}</title>
        {{{ script }}}
   </head>
   <body>
        * 必须要写，不写报错，vue-server-renderer 会自动把 html 内容插入这里
        <!--vue-ssr-outlet-->
   </body>
</html>
```	

```javascript
· server.js

....
const renderer = createRenderer({
    template: require('fs').readFileSync('./index.template.html', 'utf-8')
})

// 模板需要的东西
let temp = {
    // 对应html模板里的{{title}}
    title: 'my_blog_vue',

    // 对应html模板里的{{{script}}}，三个大括号表示不进行换行转义
    script: `
        <script src="${clientBundleFileUrl}"></script>
    `
}

· 然后在renderer.renderToString里添加个参数为temp对象

renderer.renderToString(app, temp, (err, html) => {
    res.send(html)
})



* 但是使用模板的话，需要手动在最外层添加 id = app，不然会导致客户端无法激活，例如 切换路由会刷新页面

		
    * 到 App.vue 文件里添加

    <template>
        <div id="app">
            <h1>App</h1>
        </div>
    </template>

    ....

```
<br/>

#### 十、当需要判断当前环境是开发环境还是生产环境的话可以使用如下方法

```javascript
* 在 js 文件里输入下面代码判断

console.log(process.env.NODE_ENV)

		
* 例如 vue 里需要请求转发 proxy

let http = process.env.NODE_ENV === 'production'

           ? 'http://localhost:3000'

           : '/api'


           * 判断是否是开发环境，是的话就把前缀改成 /api 

           * 如果是生产环境就改成 请求的 api 地址
```
<br/>

#### 十一、使用 [vuex](https://ssr.vuejs.org/zh/guide/data.html) 做服务端数据预渲染


> **1、** 首先安装 **vuex** 模块

```bash
npm install vuex --save
```
<br/>

> **2、** 然后再新建 `store.js` 文件
```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export function createStore () {
    return new Vuex.Store({
        state: {},
        actions: {},
        mutations: {}
    })
}
```
<br/>

> **3、** 再去 `app.js` 文件里写代码

```javascript
* 先安装 vuex-router-sync 模块

    npm install vuex-router-sync --save


* 再输入下面代码

....

import { sync } from 'vuex-router-sync'

import { createStore } from './store'


export function createApp () {
    const router = createRouter()
    const store = createStore()

    // 同步路由状态(route state)到 store
    sync(store, router)

    const app = new Vue({
        router,
        store,
        render: h => h(App)
    })

    return { app, router, store }
}
```
<br/>

> **4、** 做服务端预渲染数据

* **4.1、** 先新建一个 `api.js` 文件，输入代码

```javascript
// 使用的预渲染请求需要 Promise
export function getItem () {
    return new Promise((resolve, reject) => {

        setTimeout(() => {
            resolve(
                ["111", "222", "333"]
            )
        }, 1000)
    })
}
```
<br/>


* **4.2、** 然后再去 `store.js` 文件里添加代码

```javascript
...

import { getItem } from "./api"

export function createStore () {
    ....

    state: {
        items: []
    },

    // 这里定义方法
    actions: {

        // 这里定义的函数，需要在组件里调用
        getItem ({ commit }) {

            // 使用引入的 api 方法
            return getItem().then((item) => {

                // 使用 mutations 里的 setItem 方法，把数据存到 state 里
                commit("setItem", item)
            })
        }
    },

    // 这里定义修改数据方法
    mutations: {

        // 把数据存到 state 里
        setItem (state, item) {
            Vue.set(state, "items", item)
        }
    }
}
```
<br/>

* **4.3、** 到 `entry-server.js` 文件里添加预渲染的逻辑代码

```javascript
....

export default (context) => {
    return new Promise((resolve, reject) => {
    ....

        router.onReady(() => {
        ....

        // 加载所有的异步组件
        Promise.all(matchedComponents.map((Component) => {

            // 判断组件里是否存在 asyncData 方法
            if (Component.asyncData) {
                return Component.asyncData({
                    store,
                    route: router.currentRoute
                })
            }
        })).then(() => {

            // 把 store 里的数据赋值到 context 里，在 server.js 文件里使用
            context.state = store.state

            resolve(app)
        }).catch(reject)

        }, reject)
    })
}
```
<br/>

* **4.4、** 然后到 **Home.vue** 组件里调用

```html
<template>
    <div class="home">
        <div>{{items}}</div>
    </div>
</template>
<script>
export default {
    name: "name",

    // 使用预渲染数据方法
    asyncData ({ store, route }) {
        // 触发 action 后，会返回 Promise
        return store.dispatch('getItem')
    },

    computed: {
        items () {
            return this.$store.state.items
        }
    }
}
</script>
```
<br/>

* **4.5、** 再去 `entry-client.js` 文件里添加代码

```javascript
....

// 添加了这个代码，必须要在返回的 html 里添加服务端和客户端链接的数据
if (window.__INITIAL_STATE__) {
    store.replaceState(window.__INITIAL_STATE__)
}

router.onReady(() => {
    app.$mount("#app")
})
```
<br/>
		
* **4.6、** 再去 `server.js` 文件里添加代码

```javascript
....

// 模板需要的东西
let temp = {

    // 对应html模板里的{{title}}
    title: 'my_blog_vue',

    // 对应html模板里的{{{script}}}，三个大括号表示不进行换行转义
    script: `

        // context 里的数据是在 entry-server.js 里赋值的
        <script>window.__INITIAL_STATE__ = ${JSON.stringify(context.state)}</script>

        <script src="${clientBundleFileUrl}"></script>
    `
}

....
```

<br/>

> **5、** 做客户端预渲染数据

```javascript
· 在 entry-client.js 文件里输入代码

			
import Vue from "vue"
....

Vue.mixin({

    // 用于处理，例如，从 user/1 到 user/2 的路由时，也应该调用 asyncData 函数
    beforeRouteUpdate (to, from, next) {

        const { asyncData } = this.$options

        if (asyncData) {
            asyncData({
                store: this.$store,
                route: to
            }).then(next).catch(next)
        } else {
            next()
        }
    }
})

router.onReady(() => {

    // 1、在路由导航之前解析数据：

    // 添加路由钩子函数，用于处理 asyncData.
    // 在初始路由 resolve 后执行，
    // 以便我们不会二次预取(double-fetch)已有的数据。
    // 使用 `router.beforeResolve()`，以便确保所有异步组件都 resolve。


    router.beforeResolve((to, from, next) => {

        const matched = router.getMatchedComponents(to)
        const prevMatched = router.getMatchedComponents(from)

        // 我们只关心非预渲染的组件
        // 所以我们对比它们，找出两个匹配列表的差异组件
        let diffed = false
        const activated = matched.filter((c, i) => {
            return diffed || (diffed = (prevMatched[i] !== c))
        })

        if (!activated.length) {
            return next()
        }

        // 这里如果有加载指示器 (loading indicator)，就触发

        Promise.all(activated.map(c => {

            if (c.asyncData) {
                return c.asyncData({ store, route: to })
            }

        })).then(() => {

            // 停止加载指示器(loading indicator)

            next()
        }).catch(next)
    })

    app.$mount("#app")
})
```
<br/>

> **6、** 此时项目就可以启动了

```bash
npm run build

npm start
```
<br/>


#### 十二、浏览器热加载，用于跟服务端渲染做对比

 * 使用热加载和普通的热加载一样，写完前端代码再构建成服务端渲染

> **1、** 安装 [***webpack-dev-middleware***](https://github.com/webpack/webpack-dev-middleware)、[***webpack-hot-middleware***](https://github.com/webpack-contrib/webpack-hot-middleware)

```bash
npm install webpack-dev-middleware webpack-hot-middleware --save-dev
```
<br/>

> **2、** 然后在 **build** 目录里新建一个 `webpack.dev.js` 配置文件，用来配置热加载功能

```javascript
const path = require("path")
const merge = require("webpack-merge")
const common = require("./webpack.common.js")
const webpack = require("webpack")
const HtmlWebpackPlugin = require("html-webpack-plugin")

module.exports = merge(common, {
    entry: [
        path.resolve(__dirname, "../entry/entry-client.js"),
        "webpack-hot-middleware/client?reload=true"
    ],
    plugins: [
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoEmitOnErrorsPlugin(),
        new HtmlWebpackPlugin({
            template: path.resolve(__dirname, '../index.html')
        })
    ]
})
```
<br/>

> **3、** 然后在 **build** 目录里新建一个 `devServer.js` 文件，用来启动热加载服务

```javascript
const express = require('express')
const app = express()

const webpack = require('webpack')
const config = require('./webpack.dev')
const compiler = webpack(config)

const WebpackHotMiddleware = require('webpack-hot-middleware')
const WebpackDevMiddleware = require('webpack-dev-middleware')


app.use(WebpackHotMiddleware(compiler))
app.use(WebpackDevMiddleware(compiler))

app.listen(8080, () => {
    console.log('端口号为：8080')
})

```
<br/>

> **4、** 然后使用 [***http-proxy-middleware***](https://github.com/chimurai/http-proxy-middleware) 实现请求代理

* **4.1.** 安装 ***http-proxy-middleware*** 插件
```bash
* 不联网会报错

npm install http-proxy-middleware --save-dev

```
<br/>

* **4.2.** 再在 `devServer.js` 文件里添加代理代码
```javascript
....

const proxy = require("http-proxy-middleware")

app.use(proxy("/api", {
    target: "http://localhost:3000",    // 需要代理的地址
    changeOrigin: true,                 // 是否跨域
    pathRewrite: {
        "^/api": ""     // 本身的接口地址没有 '/api' 这种通用前缀，所以要rewrite，如果本身有则去掉 
    }
}))

....

```
<br/>


> **5、** 设置完本地环境请求跨域之后，在 **src** 目录里新建 `config.js` 文件配置请求前缀

```javascript
// 判断是开发模式的话就改成真实 api 地址

let http = process.env.NODE_ENV === "production"
           ? "http://localhost:3000"
           : "/api"

module.exports = {
    url: http
}

```
<br/>


> **6、** 然后在某个 **vue** 文件里使用  `ajax`

```javascript
import config from "./config.js"
....

mounted () {
    let xhr = new XMLHttpRequest()

    // 使用config.js文件里设置的前缀
    xhr.open("get", config.url + "/getUser")
    xhr.send()

    xhr.onreadystatechange = function () {
        if (xhr.readyState == 4) {
            if (xhr.status == 200) {
                console.log(xhr.responseText)
            }
        }
    }
}

```
<br/>

> **7、** 然后再到 **package.json** 里的 `scripts` 里添加代码

```javascript
"scripts": {
    ....

    "dev": "node build/devServer.js"
}


· 然后就可以使用npm run dev来使用浏览器热加载测试项目了
```
<br/>


#### 十三、使用 createBundleRenderer 来代替 createRenderer


* 好处：可以不需要调用 **createApp** 函数，他自身已经调用

	
> **1、** 到 `server.js` 文件里修改代码

```javascript
....

const { createBundleRenderer } = require('vue-server-renderer')


const bundle = require('./dist/vue-ssr-server-bundle.json')                 // 服务端的 json 文件
const clientManifest = require('./dist/vue-ssr-client-manifest.json')       // 客户端的 json 文件
const template = fs.readFileSync('./template/index.template.html', 'utf-8')


const renderer = createBundleRenderer(bundle, {
    template,

    runInNewContext: false,	    // 推荐配置，有性能提升

    clientManifest                  // bundle renderer 自动推导需要在 HTML 模板中注入的内容
})


server.get("*", (req, res) => {

    // 根据url获取缓存页面，如果有缓存就直接返回缓存页面
    // 针对用户点击多次刷新
    const hit = microCache.get(req.url)
    if (hit) return res.send(hit)

    const context = {
        url: req.url,
        title: 'vue-ssr-5'
    }

    renderer.renderToString(context, (err, html) => {

        if (err) {
            return handleError(err)
        }
        res.send(html)

        //将页面缓存到缓存对象中
        microCache.set(req.url, html)
    })
})

server.listen(8081, () => {
    console.log(`服务端渲染端口号为：8081`)
})


* 这样就实现了使用 createBundleRenderer 代替 createRenderer

```
<br/>


#### 十四、vue-ssr 服务端渲染的热加载


> **1、** 先去借鉴大佬的 **ssr** 热加载 [**setup-dev-server.js**](https://github.com/vuejs/vue-hackernews-2.0/blob/master/build/setup-dev-server.js) 文件，复制过来使用

```
setup-dev-server.js
```
<br/>

> **2、** 然后把里面需要修改的文件地址修改一下

```javascript
const clientConfig = require('./webpack.client')
const serverConfig = require('./webpack.server')
```
<br/>

> **3、** 然后新建一个 `dev-ssr.js` 文件，用于开启 *ssr* 热加载服务

* *3.1、* 先引入需要用到的模块

```javascript
const express = require('express')
const app = express()
const { createBundleRenderer } = require('vue-server-renderer')
```
<br/>

* **3.2、** 然后创建一个 **createRenderer** 函数，用于热加载动态刷新**createBundleRenderer** 配置

```javascript
function createRenderer(bundle, options) {
    return createBundleRenderer(bundle, Object.assign(options, {

        cache: new LRU({
            max: 1000,
            maxAge: 1000 * 60 * 15
        }),
        basedir: resolve('../dist'),
        runInNewContext: false
    }))
}
```
<br/>

* **3.3、** 然后写热加载的 **promise** 函数，作用是可以等热加载完，在 **then** 里面做后续操作
		
```javascript
let renderer
let readyPromise

const templatePath = resolve('../template/index.template.html')

// 使用热加载
readyPromise = require('./setup-dev-server')(
    app,
    templatePath,
    (bundle, options) => {

        // 每次热加载后赋值到 renderer
        renderer = createRenderer(bundle, options)
    }
)
```
<br/>

* **3.4、** 然后再写启动服务代码
		
```javascript
app.get('*', (req, res) => {
    readyPromise.then(() => {

        · 传入 url，然后 renderer 查询所匹配的路由，返回 html 代码

        const context = {
            url: req.url,
            title: 'vue-ssr-5'
        }

        renderer.renderToString(context, (err, html) => {
            if (err) return err

            res.send(html)
        })
    })
})


app.listen(8082, () => {
    console.log(`服务端渲染端口号为：8082`)
})
```
<br/>

* **3.5、** 然后在 **package.json** 里添加代码
		
```javascript
* cross-env 使用这个才可以设置环境变量

npm install cross-env --save


"scripts": {
    ....

    "dev-ssr": "cross-env NODE_ENV=development node build/dev-ssr.js"
}
```
<br/>


#### 十五、优化 node 服务器性能


> **1、** 下载 [***lru-cache***](https://github.com/isaacs/node-lru-cache) 模块

```bash
npm install lru-cache --save
```
<br/>


> **2、** 在 `server.js` 文件里添加优化代码，使用 **lru-cache** 模块

* 优化逻辑：

* 根据url获取缓存页面，如果有缓存就直接返回缓存页面

* 针对用户点击多次刷新

```javascript
....

// 删除最近最少使用条目的缓存对象
const LRU = require('lru-cache')


// 实例化配置缓存对象
const microCache = new LRU({
    max: 100,                   // 最大存储100条
    maxAge: 1000 * 60 * 15      // 存储在 15 分钟后过期
})

....

const renderer = createBundleRenderer(bundle, {
    ....

    cache: microCache		// 提供组件缓存具体实现
})


server.get("*", (req, res) => {

    // 根据url获取缓存页面，如果有缓存就直接返回缓存页面
    // 针对用户点击多次刷新

    const hit = microCache.get(req.url)

    if (hit) {
        return res.end(hit)
    }

    ....

    renderer.renderToString(app, template_data, (err, html) => {
    ....

    res.send(html)
    
        // 将页面缓存到缓存对象中
        microCache.set(req.url, html)
    })

})
```
<br/>


#### 十六、package.json 里 scripts 配置说明

```javascript
"scripts": {
    "build": cross-env NODE_ENV=production webpack --config webpack.server.js --progress --hide-modules
}
```

* cross-env NODE_ENV=production ：设置环境变量为 production（生产模式）

* --progress：设置进度条

* --hide-modules：隐藏打包文字

<br/>


#### 十七、[store 代码拆分](https://ssr.vuejs.org/zh/guide/data.html#store-%E4%BB%A3%E7%A0%81%E6%8B%86%E5%88%86-store-code-splitting)

> 说明：在大型应用程序中，我们的 **Vuex store** 可能会分为多个模块。当然，也可以将这些模块代码，分割到相应的路由组件 **chunk** 中。假设我们有以下 **store** 模块：

<br/>

> **1、** 在 **store** 文件夹里新建一个 **modules** 文件夹


> **2、** 在 **modules** 文件夹里新建一个 `foo.js` 文件，添加代码

```javascript
export default {
    namespaced: true,

    // 重要信息：state 必须是一个函数，
    // 因此可以创建多个实例化该模块
    state: () => ({
        count: 0
    }),

    actions: {
        inc ({ commit }) {
            commit('inc')
        }
    },

    mutations: {
        inc (state) {
            return state.count++
        }
    }
}
```
<br/>


> **3、** 写完单独 **stroe** 仓库，需要在单独 `.vue` 文件里引入使用


* 比如 home.vue

```html
<template>
    <div class="home">
        <h1>{{fooCount}}</h1>
    </div>
</template>

<script>

export default {

    // 在这里导入模块，而不是在 `store/index.js` 中
    import fooStoreModule from '../store/modules/foo'

    asyncData ({ store }) {

        // 惰性注册这个模块
        store.registerModule('foo', fooStoreModule)

        // 调用在 foo.js 里定义的方法
        return store.dispatch('foo/inc')
    },

    // 重要信息：当多次访问路由时，
    // 避免在客户端重复注册模块。
    destroyed () {
        this.$store.unregisterModule('foo')
    },

    computed: {
        fooCount () {
            return this.$store.state.foo.count
        }
    }
}

</script>
```
