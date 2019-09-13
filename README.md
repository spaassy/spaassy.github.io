## Spaassy教程

### 安装spaassy脚手架spaassy-cli

首先安装node，并且版本>8.6

全局安装spaassy脚手架spaassy-cli

运行node命令：

``` node
npm i spaassy-cli -g
```

初始化spaassy项目：

``` node
spaassy init -p newProject
```

init是初始化spaassy项目的命令，-p是项目名称的设置，newProject就是你项目的名称，你也可以直接执行：

``` node
spaassy init
```

没有项目名称，脚手架会默认设置新项目的名称为spaassy。

然后一路按照指示生成项目，或是直接回车使用默认设置就可以了。

spaassy框架内置了react+react-router+axios+spaassy-redux，spaassy-redux基于react-redux库并改写了内部的一些函数，在使用spaassy之前你需要参考一下spaassy相关文档。并且spaassy-redux是spaassy实现微前端的核心库。

### spaassy框架使用教程

#### 项目的目录

首先我们来看看整个spaassy应用的项目目录：

#### spaassy作为单纯的SPA应用

如果你目前只是想做一个简单单一应用，Spaassy集成了react、redux、react-redux、axios、antd... 并且在配置上尽可能的做到丰富与封装。尽可能的让适用者专注于业务逻辑的开发，而不是把精力放在环境和各种依赖库的整合上面。

只是spaassy内置了spaassy-redux，这是spaassy中较为核心的地方，在使用redux的时候会与react-redux有一点点不一样。

在通过脚手架生成项目之后，我们进入项目目录当中：

``` node
cd newProject
```

然后安装依赖：

``` node
npm install
```

src文件夹就是管理我们项目的业务逻辑的文件，和其它项目不同的是，spaassy生成的工程多了三个配置文件：

* _spaassyConfig.js；

这个文件是用来配置本地开发的文件，包括本地服务端口号、proxy代理、文件alias，还有webpackDll公用依赖打包的配置。例如：

``` javaScript
const path = require('path')

module.exports = {
    server: {
        host: '127.0.0.1',
        port: 9001,
        proxy: [{
            path: '/api',
            option: {
                target: 'http://127.0.0.1:8989',
                pathRewrite: {
                    '^/api': "/"
                },
                changeOrigin: true
            }
        }]
    },
    webpack: {
        env_variable: {
    'process.env.PROJECTTYPE': JSON.stringify('SPAASSY'), // 系统级别，独立作为spa应用设置为“SPA”, 作为spaassy应用设置为“SPAASSY”
    'process.env.SYSTEMNAME': JSON.stringify('main') // 系统名称，会被作为系统的命名空间, 自定义命名
},
        alias: {
            '@': path.resolve(__dirname, 'src'),
            '@assets': path.resolve(__dirname, 'src/assets'),
            '@http': path.resolve(__dirname, 'src/httpServer'),
            '@views': path.resolve(__dirname, 'src/views'),
            '@common': path.resolve(__dirname, 'src/common'),
            '@store': path.resolve(__dirname, 'src/store')
        },
        // 将第三方依赖单独打包并作为外部依赖，请保持主系统与系统的vendor配置一致
        dll: {
            entry: {
                vendor_dll: [
                    'react',
                    'redux',
                    'react-redux',
                    'axios',
                    'react-router',
                    'react-router-dom'
                ],
            }
        }
    }
}
```

* _spaassyportalConfig；

portalConfig是将当前项目作为一个门户系统打包，需要配置的，它的内部是长这个样子的：

``` javascript
const portal = {
    portalTarget: './dist/index.html',
    subProject: [{
        projectName: 'mainSub',
        host: './',
        target: './distSub/index.html',
        resourcePattern: ['mainSub/main.css', 'mainSub/bundle.js']
    }]
}

module.exports = portal
```

* _spaassySubConfig；

_spaassySubConfig是将当前项目作为一个微前端子系统打包，需要配置的，它的内部是长这个样子的：

``` javaScript
import {
    SpaAssyRegister
} from 'spaassy-redux'
import reducers from '@store'
import routers from '@/views/home/routers'

const namespace = process.env.SYSTEMNAME + 'Sub'

let option = {
    namespace: namespace,
    routers: [...routers],
    reducers: {
        ...reducers
    }
}
const spaassyRegister = new SpaAssyRegister(option)

spaassyRegister.addRouters()
spaassyRegister.registerReducer()
```

当然，如果你只是将spaassy工程为一个独立的SPA项目来使用，_spaassyportalConfig与_spaassySubConfig这个两个配置文件你完全不用理会和使用。

执行：

``` javaScript
npm run start
```

项目启动。

为了加速编译和共享第三方依赖，spaassy-cli内置了webpackDll，你只要分别执行：

开发环境dll

``` javaScript
npm run dllDev
```

生产环境dll

``` javaScript
npm run dllPro
```

然后，
执行：

``` javaScript
npm run start
```

就可以了。

如果你不想使用Dll，运行

``` javaScript
npm run clearDll
```

就可以了。

#### spaassy作为微前端应用

1、 首先初始化一个工程作为门户系统：

``` javaScript
spaassy init - p portal
```

在_spaassyConfig文件中配置项目名称为portal：

``` javaScript
env_variable: {
    'process.env.PROJECTTYPE': JSON.stringify('SPAASSY'), // 系统级别，独立作为spa应用设置为“SPA”, 作为spaassy应用设置为“SPAASSY”
    'process.env.SYSTEMNAME': JSON.stringify('portal') // 系统名称，会被作为系统的命名空间, 自定义命名
},
```

名称可以自定义。

2、 初始化一个子系统：

``` javaScript
spaassy init - p subProject
```

在_spaassyConfig文件中配置项目名称为subProjec：

``` javaScript
env_variable: {
    'process.env.PROJECTTYPE': JSON.stringify('SPAASSY'), // 系统级别，独立作为spa应用设置为“SPA”, 作为spaassy应用设置为“SPAASSY”
    'process.env.SYSTEMNAME': JSON.stringify('subProjec') // 系统名称，会被作为系统的命名空间, 自定义命名
},
```

名称可以自定义。

两个文件我们直接使用示例代码。

3、 在portal项目中打开，入口文件“src/index.jsx”：

``` javaScript
import React from 'react';
import ReactDom from 'react-dom';
import App from '@views/app';
import {
    SpaAssyProvider
} from 'spaassy-redux'
import 'lodash'
import './common';

import rootReducers from '@store'

const appEle = document.getElementById('app');
const namespace = process.env.SYSTEMNAME

ReactDom.render( 
    <SpaAssyProvider 
    rootReducers = {
        rootReducers
    }
    namespace = {
        namespace
    }
    mainProject >
    <App / >
    </SpaAssyProvider>,
    appEle
);

if (module.hot) {
    console.log('启用热加载更新！')
    module.hot.accept();
}
```

在SpaAssyProvider 组件中配置 mainProject 属性，表示当前项目作为一个portal系统。

4、 在subProject 中，配置_spaassySubConfig.js 文件把当前系统要集成到portal系统中的路由和reducer使用spaassy-redux 工具注册进去：

``` javaScript
import {
    SpaAssyRegister
} from 'spaassy-redux'
import reducers from '@store'
import routers from '@/views/home/routers'

const namespace = process.env.SYSTEMNAME

let option = {
    namespace: namespace,
    routers: [...routers],
    reducers: {
        ...reducers
    }
}
const spaassyRegister = new SpaAssyRegister(option)

spaassyRegister.addRouters()
spaassyRegister.registerReducer()
```

5、 在portal项目中通过spaassy-redux 提供的方法 SpaAssyRegister 来获取子项目的router 并注入到portal中.

src/views/home/routers.js:

``` javaScript
import { AsyncComponent } from 'spaassy-redux'
import rootRouters from '@views/rootRouters'
import {
    SpaAssyRegister
} from 'spaassy-redux'

const spaassyRegister = new SpaAssyRegister()

const subRouters = spaassyRegister.getRouters()
let subRouterList = []
Object.keys(subRouters).map(o => {
    subRouterList.push(...subRouters[o])
})

const routers = [
    // ...rootRouters,
    ...subRouterList
]

export default routers
```
为了查看效果，我们先把portal本身的路由注释掉。

6、 打包subProject项目：
``` javaScript
npm run buildSub
```

7、 打包portal项目
``` javaScript
npm run build
```

8、 我们把子系统distSub包里的文件拷贝一份到portal系统中，把distSub包的子系统资源包subProjec拷贝一份到dist里，并配置portal里的_spaassyportalConfig：

``` javaScript
// example
const portal = {
    portalTarget: './dist/index.html',
    subProject: [{
        projectName: 'subProject',
        host: './',
        target: './distSub/index.html',
        resourcePattern: ['subProject/common.*.chunk.js', 'subProject/main.js']
    }]
}

module.exports = portal
```

运行发布命令：

``` javaScript
npm run portalPublish
```

点击运行portal工程里dist文件下的index.html， 子系统的路由已经注入到了portal中。

示例代码请参照：https://github.com/spaassy/example.git
