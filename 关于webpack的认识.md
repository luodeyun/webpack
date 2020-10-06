# 关于webpack的认识

#### 一、什么是webpack

-  webpack是一个模块打包器
- 在webpack看来，前端的所有支援文件（js，css等）都会作为一个模块处理
- 他将根据模块的依赖关系进行静态分析，生成对应的静态资源

#### 二、五个核心概念

- Entry:入口文件 （entry point）指示 webpack应该使用哪个模块，来作为构建其内部依赖图的开始
- Output：output属性告诉webpack在哪里输出他所创建的bundles ，以及如何命名这些文件，默认值为/dist
- Loader:loader让webpack能够处理那些非javaScript、json文件
- Plugins：插件则可以用于执行范围更广的任务，插件的范围包括，从打包到优化和压缩，一直到重新定义环境中的变量
- Mode：模式有2个  生产模式：production 和开发模式 development

#### 三、开启项目

这时我们应该开启一个项目生成package.json文件   也就是 npm init

然后安装webpack        执行命令： npm install webapck webapck-cli   -D  (本地安装)

结论：webpack能够变异打包js和json文件，能将es6的模块化语法器能浏览器识别的语法，能压缩代码

缺点：不能编译打包css，img文件

​             不能将js的es6基本语法转化为es5

在项目的根目录下生成配置文件**webpack.config.js**

##### 指定入口出配置

```
const {resolve} = require('path')
module.export={


  entry:'./src/js/index.js'    //指定入口
  output:{
          path:path.resolve(__dirname,'dist') //输出路径 要映入path
          filename:'myfirst-webapck.bundle.js' //输出的文件名
         }
  mode : {
          'production'    //生产依赖
         }
}
```

到了这一步你可以总结运行项目然后会生成打包文件  

但是如果你想引入样式   你需要借助**loader**

先下载loader：

```
npm install css-loader style-loader less-loader
```

##### 配置样式

```
const {resolve} = require('path')
module.export={


  entry:'./src/js/index.js'    //指定入口
  output:{
          path:path.resolve(__dirname,'dist') //输出路径 要映入path
          filename:'myfirst-webapck.bundle.js' //输出的文件名
         }
  module:{
          rules:[
                     {
                      test: /.\less$/    匹配所有的文件
                      use:[     //这是从下到上加载的，+++
                             'style-loader', // 然后创建ainstyle标签，添加上js中的代码
                              "css-loader",  //将css以commonjs文件的方式整合到js文件中
                             "less-loader", // 将less转化成css文件，在内存里
                          ]
                     }
                ]
         }
  mode : {
          'production'    //生产依赖
         }
}
```

##### **语法检查**

概述 ：对js基本语法错误、隐患，进行提前检查

 安装loader：

```
npm install eslint-loader  eslint -D
```

在webpack.config.js文件中的rules数组中添加

```
    rules:[ 
     ...
                       {
                      test: /.\less$/    匹配所有的文件
                      use:[     //这是从下到上加载的，+++
                             'style-loader', // 然后创建ainstyle标签，添加上js中的代码
                              "css-loader",  //将css以commonjs文件的方式整合到js文件中
                             "less-loader", // 将less转化成css文件，在内存里
                          ]
                     },
                     {
                     test:/.\js$/,
                     exclude:'node-modules' ,   //不检查node-modules
                     loader : "eslint-loader",
                     }
                ]
```

其余的规则在package.json中配置

```
"eslintConfig":{
     "parserOption":{
         "ecmaVersion":6   //支持es6
         "SourceType":"module", //使用es6模块化
     },
     "env":{ //配置环境
           "browser":true,
           "node":true
     
     },
     "global":{ //声明全局变量
              "$" : "readonly",
              "Promise":"readonly",//声明可读
     },
     "rules":{  //eslint检查的规则 0忽略 1警告 2错误
              "no-console":0,
              "eqeqeq":2,
              "no-alert":2    
     },
     "extends":"eslint:recommended" //使用eslint推荐的默认规则
 }

```

##### 语法转换

概述：将浏览器不能识别的新语法转换成原来识别的旧语法并作兼容性处理

安装loader

```
npm install babel-loader  @babel/core @babel/preset.env   core.js -D
```

配置loader

```
rules:[
      ...
      {
      test:/.\js$/
      exclude:/node-module/,
      use:{
            loader:'babel-loader'
            options:{
                   presets:[
                             '@babel/preset-env',
                             userBuitIns:'usage'  //按需引入加载  
                             core.js:{version:3}  //解决不能找到core.js
                             targets:{  指定兼容处理那些浏览器
                                 "chrome":"58", 
                                 "ie" : '9'
                             }
                   ]
            
            }
          }
      }
 
]
```



##### 打包样式中的图片资源

概述：图片文件webpack不能解析，需要借助loader解析

安装loader    

```
npm install file-loader url-loader -D
```

url-loader和fileloader差别在于前者可以base64对图片进行处理

```
rules:[
     ...
      {
      test:'.(png|jpg|gif)'，
      use :[
            loader:'url-loader',
            options:{
            limit:8192    //8kb一下会base64处理
            outputpath:'images' //决定文件本地输出路径
            publicpath:'../build/images',
            name:'[hash:8].[ext]',//修改文件名称
            }
           ]
      }
]
```

##### 打包html文件

概述：html文件webapck不能解析，需要借助插件编译解析

安装plugins

```
npm install html-webapck-plugin -D
```

在webpack.config.js中引入插件

```
const HtmlWebpackPlugin = require('html-webpack-plugin')
```

配置插件

```
plugins:[
new HtmlWebpackPlugin(
{
template:'./src/index.html' //以当前为那件为模板创建新的html（结构和原来一样）
 }
)
]
```

打包html中的图片资源

概述：html中的图片url-loader没法处理，他在你那个处理js中引入的图片、样式中的图片，不能处理HTML中的img标签，需要引入其他的html-loader

安装loader

```
npm install html-loader -D
```

配置loader

```
rules:[ 
         ...
      { 
       test :/.\(html)$/,
       use:{
       loader:'html-loader'
       }
      }
]
```

##### 打包其他资源

概述：其他资源webpak不能解析，需要借助loader解析，比如一些字体图标，音频等

他是基于file-loader

```
{
test:'/.\(eot|svg|woff|woff2|ttf|mp3|mp4|)$/',
loader:'file-loader',
options:{
outputpath:'media',
name:'[hash:8].[ext]'
}
}
```

##### 自动编译打包运行

安装loader

```
npm install webpack-dev-server -D
```

在webpack.config.js中配置

```
devserve:{
   open:true, //自动打开浏览器
   compress:true, //启动z-gip压缩
   port:3000
}
```

这里有个坑，你会发现找不到  图片了

这时你需要配置图片路径在url-loader中：publicpath  :    ' images/'

你还可以修改package.json中的指令

```
start：webpack-dev-serve
```

##### 热模块替换

在devserve加一个配置

```
devserve:{
   open:true, //自动打开浏览器
   compress:true, //启动z-gip压缩
   port:3000
   hot:true
}
```

样式支持热莫替换，结构不支持，如果结构变了，页面是不知道的，因为wepack只监视index.js的变化，所以你要加个入口

devtool：一种将压缩文件映射回源文件的技术

devtool:'cheap-module-eval-source-map'

​		





