
### webpack进阶知识 ###

> 1、安装node环境<br/>
> 2、```npm init ``` 初始化一个package.json<br/>
> 3、终端中输入 ``` npm i webpack webpack-cli -D ```<br/>
> 4、目录下创建src/index.js <br/>
> 5、打包方式：<br/>
        1、```npx webpack ```<br/>
        2、```npx webpack --config webpack.xxx.xxxx.js(自定义webpack文件)```<br/>
        3、" build": "webpack " / " build": "webpack --config webpack.xxx.xxx.js(自定义webpack文    件) " （在package.json中配置 通过npm run build来进行打包==>常用）<br/>
 >     6、热更新  ```npm i webpack-dev-server -d   ```<br/>
 >     7、html插件 ``` npm i html-webpack-plugin -D``` <br/>
### webpack核心概念 ###
        1：入口（entry）
        2：输出（output）
        3：(loader)
        4:插件(plugins) 解决loader无法解决的事情
#### 创建webpack.config.js来配置webpack      
```js
//webpack基础配置 webpack.config.js
const path = require('path')
module.exports = {
    entry: './src/index.js',//入口文件
    output: {
        //path.resolve解析当前的相对路径为绝对路径
        path: path.resolve('./dist/'),
        filename: 'main.js',
    },
    mode: 'development'//production:上线（压缩混淆） development:开发
    devServer: { //热更新
        open: true,//开启
        compress: true,//压缩
        prot: 3000,//端口号
        contentBase: './src',//目录
        hot: true//热更新
    }
}
```
