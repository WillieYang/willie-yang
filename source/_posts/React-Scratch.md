---
title: React前端开发框架的搭建以及相关类库的选择与使用
date: 2019-06-17 20:51:20
tags: 
- React
- Front End
---

## 主要技术栈

### 基础框架

- React

### 脚手架

- create-react-app
  - 使用的是CRA 2.0版本，没有使用npm run eject导出相应的webpack配置，因为这个过程是不可逆的，所以暂时对这个操作持保留态度。
    - 如果想用webpack多入口，然后打包成多页面应用，需要注意的是，这个多页面应用里面，每个页面都必须是一个单独的React app，同时也无法是用react-  router，使用redux进行状态管理也很麻烦。

### 路由

- react-router
  - 使用的是react-router的最新版本。

### UI框架

- Ant Design
  - 这个组件库内置了很多通用组件，比如说表格，菜单栏，走马灯，以及树形控件，图标等等，可以减轻研发和UI设计的工作量。
  - 里面有一个按需加载的设置，官方推荐配合react-app-rewired和customize-cra进行相应的配置，之后就可以实现组件的按需加载。一般来说，在设置之前，使用import {Button} from 'antd'; 会引入antd下面所有的模块，但是在babel-plugin-import的加持下，可以实现antd组件的按需加载，这个插件会自动将import方式转换成antd/lib/xxx的写法。

### 状态管理

- Redux
  - 独立的状态管理工具，拥有actions, reducers, state来对状态进行相应的触发，修改和管理。
- react-redux
  - 用于连接组件和Redux应用，它提供了Provider和connect这两个函数(组件)，用于设定组件的state和props，并和redux中的creaeStore函数配合，根据使用者的行为改变存在于store中的state. 其中，connect这个函数接收两个参数，分别是mapStateToProps和mapDispatchToProps. 前者负责将store里面的state映射到组件的props里面，后者负责将用户对组件的操作映射成Action，并传入到dispatch函数里面.
- redux-thunk
  - 作为一个redux的异步中间件，可以对异步操作进行处理，主要的原理是让action creator返回一个函数（之前只能返回一个action对象）。
  - 当action creator被传入dispatch函数以后，thunk这个中间件会对其进行判断，如果传入的action creator是一个action对象，就会直接分配给相应的reducer进行state的计算，如果是一个函数，就会直接执行这个函数，并且dispatch和getState这两个函数都会当作参数传入函数中，这个函数就可以用来对异步http request进行处理，然后根据传入的dispatch方法，手动对action进行触发，此时action的这个对象里面就能够用来放置从后端请求到的值，并在reducer中对其进行处理，并以此为基础返回需要的数据。

### HTTP库    

- Axios
  - 通过这个http库，创建一个API工具类，根据后端接口的base url，new一个API实例并将其导出，在这个实例里面可以对http request进行相应的配置，比如说添加相应的header.

```json
"dependencies": {
    "antd": "^3.16.5",
    "axios": "^0.18.0",
    "echarts": "^4.2.1",
    "echarts-for-react": "^2.0.15-beta.0",
    "lodash": "^4.17.11",
    "lodash-es": "^4.17.11",
    "node-sass": "^4.12.0",
    "prop-types": "^15.7.2",
    "react": "^16.8.6",
    "react-dom": "^16.8.6",
    "react-redux": "^7.0.2",
    "react-router-dom": "^5.0.0",
    "react-scripts": "2.1.8",
    "redux": "^4.0.1",
    "redux-thunk": "^2.3.0",
    "styled-components": "^4.2.0"
  },
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-app-rewired eject"
  },
  "devDependencies": {
    "babel-eslint": "^9.0.0",
    "babel-plugin-import": "^1.11.0",
    "customize-cra": "^0.2.12",
    "eslint": "^5.12.0",
    "eslint-config-airbnb": "^17.1.0",
    "eslint-plugin-import": "^2.17.2",
    "eslint-plugin-jsx-a11y": "^6.2.1",
    "eslint-plugin-react": "^7.12.4",
    "react-app-rewired": "^2.1.3"
  }
```



## 目录结构

{% asset_img folder.png Project Structure %}

### public
public文件夹里面包含了HTML文件，我们可以对其进行调整，比如说设置页面标题。除此之外，也可以将其他静态资源或者配置文件放入其中，Webpack将不会对这个文件夹中的文件进行打包处理。

### src
src文件夹包含了整个工程的源代码。

### App
App文件夹包含了整个项目的总体布局配置，导航路由配置，以及统一的scss样式修改。

### Layout 
Layout文件夹主要包含了整个工程的页面布局，包括header(页头)，Navigation(导航)，Content(内容)和Footer(页脚)这几个部分，都是分开的组件，可以进行相应的定制工作。

### Pages
Pages文件夹主要包含了整个工程的页面布局下，根据导航栏的路由跳转，从而渲染出来的不同的Content的内容。相应的功能组件都能够写在这个文件夹下的不同页面里面，进行相应的组件拆分和复用。

### components
components文件夹主要包含了整个工程的可复用的组件，可以根据不同业务进行相应的目录拆分。<br />关于组件的命名风格：<br />扩展名：React组件使用.jsx扩展名。<br />文件名：文件名使用帕斯卡命名。 例如： ReservationCard。同时，在这个文件里面统一使用index.jsx的命名方式默认导出相应组件。

### stores
stores文件夹主要包含了redux状态管理的相关代码，例如state, actions, reducers等等，其中state，reducers, actions都可以根据不同组件进行相应的拆分，同时action里面的type由于是常量字符串，也可以拆分到相应的constant文件里面。 

### utils 
utils文件夹主要包含了可以通用的工具类，比如说和后台交互的通用的API等等.

## 其他相关类库

### ESLint
ESLint是一个用来识别ECMAScript，并根据规则给出报告的代码检测工具，主要用于避免低级错误，以及统一代码风格，该工作应选择在系统开发初期进行，在这个项目工程下，我选择使用了airbnb的代码规范，并且根据实际情况，修改了相应的rule。

### echarts-for-react
ECharts是一个基于JavaScript的数据可视化图标库，echarts-for-react则是基于react框架下的echarts的相应实现，可以让开发者以组件的方式渲染出相应的图表。

### node-sass
node-sass是一个提供Node.js和LibSass绑定的类库，它能够快速的将.scss文件编译成css文件，在这个类库的帮助下，整个工程可以使用scss来进行统一样式处理。

### prop-types
prop-types是React官方推荐的第三库，里面有一整套验证器，主要用于react组件里面，props对象中的变量进行类型检查，可以避免一些低级错误，当给属性传递了无效值时，JavaScript控制台会打印出相应的警告，但是只会用于开发模式下进行相应的检查。

### react-app-rewired和customize-cra
react-app-rewired是一个React社区用来修改CRA配置的工具，它提供了一个config-overrides.js文件，用于修改webpack的配置。CRA升级到2.0以后，react-app-rewired需要和customize-cra搭配使用。

### styled-components
styled-components是一个基于CSS in JS 方式实现的一个库。它能够让样式写成组件的形式，实现在JS上编写CSS，也可以使用JS的语法来封装CSS对象，或者通过参数来动态定义样式。

### Lodash-es/Lodash
Lodash是一个JavaScript类库，它能够提供一些内置的工具函数来完成一些编程任务，采用的是函数式编程思想。<br />


## 版本控制

### Git
使用Git进行版本控制<br />分支：master/dev。根据开发人员酌情增加开发人员的个人分支<br />需要对开发的不同阶段进行tag标记。