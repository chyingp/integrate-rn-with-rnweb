## 一、写在前面

react-native-web 的基本原理，就是将 react-native 的组件，针对web的场景从新实现一遍。借助构建工具，可以实现 react-native 一套代码，多端运行（终端 + web端，实际并没有那么简单）。

很多同学比较关心的是，对于现有的 RN 项目，如何将 react-native-web 整合进去，下文会通过简单的例子逐步进行说明。

## 二、新建RN项目

下面例子来自[官方文档](https://facebook.github.io/react-native/docs/getting-started)，经过一定程度的简化，建议查看原文档。

首先安装依赖：

```bash
# 注释后面的是笔者本地安装的版本
brew install node # 10.10.0
brew install watchman # 4.9.0
npm install -g react-native # 0.61.3
```

初始化RN项目，并安装依赖：

```bash
react-native init RnWebTest
cd RnWebTest
npm install
```

在ios模拟器里运行，搞定。

```bash
npx react-native run-ios
```

下面开始讲解如何整合 react-native-web。

## 三、react-native-web环境准备

react-native 有自己的构建打包工具，针对 react-native-web 需要自己搞一套，同样是webpack + babel全家桶。

### 依赖安装

首先，安装核心依赖 react-native-web：

```bash
npm install --save react-native-web
```

其次，安装babel及相关 preset / plugin：

```bash
npm install --save-dev @babel/core
npm install --save-dev @babel/runtime
npm install --save-dev @babel/preset-env
npm install --save-dev @babel/preset-react
npm install --save-dev @babel/preset-flow
```

安装webpack及相关loader：

```bash
npm install webpack webpack-cli webpack-dev-server --save-dev
npm install --save-dev babel-loader
```

### babel配置

 .babelrc如下：

```json
{
  "presets": [
  	[
  		"@babel/preset-env", {
  			"modules": "commonjs"
  		}
  	],
  	"@babel/preset-react",
  	"@babel/preset-flow"
  ]
}
```

### webpack配置

webpack.config.js 如下：

```javascript
const path = require('path');

module.exports = {
	entry: './index.web.js',
	output: {
		filename: 'index.web.js',
		path: path.resolve(__dirname, 'build'),
	},
	mode: 'development',
	module: {
		rules: [
			{
				test: /\.js$/,
				exclude: /(node_modules|bower_components)/,
				use: {
					loader: 'babel-loader',
					// options: {
					// 	presets: ['@babel/preset-env']
					// }
				}
			}
		]
	},

	resolve: {
		alias: {
			'react-native$': 'react-native-web'
		}
	},

	devServer: {
		contentBase: path.join(__dirname, '.'),
		// compress: true,
		port: 9000
	}
}
```

最重要的就是这几行：

```javascript
resolve: {
	alias: {
		'react-native$': 'react-native-web'
	}
}
```

### NPM 脚本

添加如下脚本：

```
"scripts": {
  "dev": "webpack -w",
  "build": "webpack"    
},
```

## 四、业务代码修改

上述环境准备好后，直接 `npm build` 会报错，需要对业务代码进行微调，具体如下。

### App.js兼容修改

经过上述修改后，构建的时候会报错，因为 App.js 中引用了 react-native 中的库文件 NewAppScreen，而 NewAppScreen 在 react-native-web 中并不存在。

这里图省事，直接把不支持的代码注释掉，包括组件使用的地方。

```javascript
// import {
//   Header,
//   LearnMoreLinks,
//   Colors,
//   DebugInstructions,
//   ReloadInstructions,
// } from 'react-native/Libraries/NewAppScreen';
```

样式的定义用到了 Colors 变量，这里直接把定义拷贝过来：

```javascript
const Colors = {
  primary: '#1292B4',
  white: '#FFF',
  lighter: '#F3F3F3',
  light: '#DAE1E7',
  dark: '#444',
  black: '#000',
};
```

### 新建入口文件index.web.js

首先，创建入口文件 index.web.js，跟 RN 的入口文件 index.js区分开。

相比 index.js，多了 `AppRegistry.runApplication()` 这行调用。

```javascript
import {AppRegistry} from 'react-native';
import App from './App';
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => App);

// 多了这行
AppRegistry.runApplication(appName, {
    initialProps: {},
    rootTag: document.getElementById('root')
});
```

## 五、运行应用

首先，启动本地构建：

```bash
npx webpack-dev-server
```

浏览器中访问 http://127.0.0.1:9000/index.html，搞定。



## 六、参考链接

https://facebook.github.io/react-native/docs/getting-started
https://github.com/necolas/react-native-web