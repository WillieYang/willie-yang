---
title: React中如何结合React Router以及Token对登陆权限进行控制
date: 2019-06-18 20:17:02
tags:
- React
- Token
---

## 前言

对于所有的Web应用，登录和注册功能的实现都是最重要的部分，它们既是是网站的入口，也在同时控制了用户的权限，在使用React框架实现登陆注册功能的时候，又该如何对登陆权限进行控制呢？这篇文章将会使用react-router v4对路由进行控制, 与此同时，结合cookie对后端传入token进行控制，来进行登陆权限的管理。

## 私有路由

Route是react-router-dom里面最为重要的一个组件，它可以在路由地址匹配上时，用来渲染相应的UI组件，正如下图所示：

``` html
    <Route path="/onlineMap" component={OnlineMap} />
    <Route path="/newsCenter" component={NewsCenter} />
    <Route path="/resourceCenter" component={ResourceCenter} />
    <Route path="/userCenter" component={UserCenter} />
```

而我们的目的，就是重写这个Route组件，从而生成一个私有路由组件，并在这个私有组件里面，根据我们自己的需求，有条件的渲染相应的组件。私有路由组件如下图所示：

``` js
const PrivateRoute = ({ component: Component, ...rest }) => ({
  render() {
    return (
      <Route
        {...rest}
        render={props => (
          cookies.get('access_token')
            ? <Component {...props} />
            : <Redirect to="/login" />
        )}
      />
    );
  },
});
```
上图的PrivateRoute组件会接收从外部传进来的Component，我们这里使用了ES6 Spread语法，将Component单独提取出来，PrivateRoute这个组件会返回一个Route组件，在这个Route组件里面，我们通过判断浏览器的cookie里面是否存有token，从而判断是否渲染出相应的组件，亦或者使用react-router-dom里面的Redirect组件，将页面跳转到login页面。

(*Tips: React router里面的render prop也可以用来返回组件，和component prop的区别只是后者在每次路由匹配成功渲染相应组件的时候, 会执行该组件里面所有的生命周期函数，而前者不会。*)

下图就是PrivateRoute组件的用法，通过这种方式，就可以在路由层面，对相应的用户权限进行管理，同理，也可以重写出对用户角色进行控制的Route组件。

```html
<Route path="/threeDimentionEarth" component={ThreeDimentionEarth} />
<PrivateRoute path="/helpCenter" component={HelpCenter} />
<Route path="/login" component={Login} />
```

## Token管理

至于对token的管理，这个值应该是在用户登录成功时，由前端或者后端将其存入cookie，并在用户登出时，将其移除。至于我为什么要使用token，而不是React里面的state来保持登陆状态，是因为后者会在刷新的时候恢复成默认值，而token会被保存在cookie里面(*也可以用localStorage或者sessionStorage来保存*)，除非被清除或者重新设置，不然不会改变。

## 对HTTP请求header的动态加载

有的时候，在登录成功以后，需要将token添加到请求头header里面，然后在登录状态下的所有后台请求里面，都需要加上和token相关的header，如果退出登录状态，则token相关的header则需要从请求头header里面移除，只有这样，才能保证应用的安全性。

在这里，我的HTTP库使用的axios，因此，可以使用axios里面的拦截器(interceptor)在请求发起，但是还未被`then`或者`catch`处理的时候，对里面的config进行相应的判断处理，动态添加token的header.

``` js
const testAPI = axios.create({
  baseURL: 'http://httpbin.org/',
  responseType: 'json',
  headers: {
    'X-Auth-Token': accessToken,
    'Accept-Language': 'zh-CN',
    'Content-Type': 'application/json; charset=UTF-8',
  },
});

testAPI.interceptors.request.use((config) => {
  if (cookies.get('access_token')) {
    console.log(cookies.get('access_token'));
    accessToken = cookies.get('access_token');
  } else {
    accessToken = 'Not_existed';
  }
  const configAxios = config;
  configAxios.headers['X-Auth-Token'] = accessToken;
  return configAxios;
}, error => Promise.reject(error));
```

如上图所示，根据cookie里面的token进行条件判断，就可以实现对HTTP请求header进行动态加载了。
