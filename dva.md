### DVA的基本使用
###### dva目录介绍
```js
/*
mock       ---->后端没有提供接口的时候，模拟数据
assets     ---->静态资源
components ----> 通用组件
models     ---->redux部分
routers    ---->url中路由的页面   
services   ---->接口
*/
```
### <font face='宋体' color="#666">路由跳转 </font> 
``` jsx  
//userPage.js
import React, { Fragment } from 'react';
import { Link } from 'dva/router'
import { connect } from 'dva';
import Child from '../components/Child.js'

class userPage extends React.Component {
  handleToIndex = () => {
    console.log(this.props);
    /*
    history: {length: 2, action: "POP", location: {…}, createHref: ƒ, push: ƒ, …}
    location: {pathname: "/user", search: "", hash: "", state: undefined}
    match: {path: "/user", url: "/user", isExact: true, params: {…}}
    */
    this.props.history.push('/')//跳转到首页
  }
  render() {
    return (

      <Fragment>
        <div>111</div >
        <button onClick={this.handleToIndex}>首页</button>
        <Child />
      </Fragment>
    )
  }
}

export default connect()(userPage);

//Child.js---->通用组件

import React from 'react';
import { withRouter } from 'dva/router';
class Child extends React.Component {
  handleToIndex = () => {
    console.log(this.props);//直接拿拿不到props  需要通过 withRouter写入一个路由，导出的时候包裹当前组件 export default withRouter(Child）
    this.props.history.push('/')//跳转到首页
  }
  render() {
    return (
      <div>
        <div>我是通用组件</div>
        <button onClick={this.handleToIndex}>Child</button>
      </div>
    )
  }
}

export default withRouter(Child);


```
### reducers
###### router     IndexPage.js
```js

import React from 'react';
import { connect } from 'dva';

class IndexPage extends React.Component {
  changeName = () => {
    this.props.dispatch({
      type: 'indexTest/changeName',//传入命名空间/定义的操作函数
      //会打印到IndexTest.js中reducers的changeName的方法
      data: {
        name: 'zhuzhuzhu'
      }
    })
  }
  render() {
    /**
     * console.log(this.props);
     * dispatch: ƒ (action)
       history: {length: 1, action: "POP", location: {…}, createHref: ƒ, push: ƒ, …}
       location: {pathname: "/", search: "", hash: "", state: undefined}
       match: {path: "/", url: "/", isExact: true, params: {…}}
       msg: "好好学习、天天向上"
       name: "zs"
       staticContext: undefined
     */

    return (
      <>
        <div>111</div>
        <p>{this.props.msg}</p>
        <p>{this.props.name}</p>
        <button onClick={this.changeName}>changeName</button>
      </>
    )
  }
}
const mapstateToProps = state => {
  /**
   * console.log(state);  ！！！会接受到models中state的数据！！！
   * indexTest: {name: "zs"}
     routing: {location: null}
   */
  return {
    msg: '好好学习、天天向上',
    name: state.indexTest.name
  }
}
export default connect(mapstateToProps)(IndexPage);

```
###### modules     IndexTest.js
```js
export default {
  namespace: 'indexTest',
  state: {
    name: '张三',
    age: 18
  },
  reducers: {
    changeName(state, payload) {
      /**
      //  * console.log(payload, 'run'); playload会接收到显示页面中传过来的数据
       * data: {name: "zhuzhuzhu",age : 18}
         type: "indexTest/changeName"
       */
      // console.log(state);当前页面的state
      state.name = payload.data.name  //将state.name赋值为显示页面更改的数据
      let _state = JSON.parse(JSON.stringify(state))
      return _state
    }
  },
  effects: {
    *changeNameAsync({ payload }, { put, call }) {
      yield put({
        type: 'changeName',
        data: {
          name: '超人强'
        }
      })
    }
  }
}

```
