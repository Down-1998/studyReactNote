# dva

> 官方网站：https://dvajs.com
> dva不仅仅是一个第三方库，更是一个框架，它主要整合了redux的相关内容，让我们处理数据更加容易，实际上，dva依赖了很多：react、react-router、redux、redux-saga、react-redux、connected-react-router等。



# dva的使用

1. dva默认导出一个函数，通过调用该函数，可以得到一个dva对象
   
2. dva对象.router：路由方法，传入一个函数，该函数返回一个React节点，将来，应用程序启动后，会自动渲染该节点。

3. dva对象.start: 该方法用于启动dva应用程序，可以认为启动的就是react程序，该函数传入一个选择器，用于选中页面中的某个dom元素，react会将内容渲染到该元素内部。

4. dva对象.model: 该方法用于定义一个模型，该模型可以理解为redux的action、reducer、redux-saga副作用处理的整合，整合成一个对象，将该对象传入model方法即可。
   1. namespace：命名空间，该属性是一个字符串，字符串的值，会被作为仓库中的属性保存
   2. state：该模型的默认状态
   3. reducers: 该属性配置为一个对象，对象中的每个方法就是一个reducer，dva约定，方法的名字，就是匹配的action类型
   4. effects: 处理副作用，底层是使用redux-saga实现的，该属性配置为一个对象，对象中的每隔方法均处理一个副作用，方法的名字，就是匹配的action类型。
      1. 函数的参数1：action
      2. 参数2：封装好的saga/effects对象
   5. subscriptions：配置为一个对象，该对象中可以写任意数量任意名称的属性，每个属性是一个函数，这些函数会在模型加入到仓库中后立即运行。

   ## index.js

```javascript
import React from 'react';
import App from "./App"
import dva from "dva";
import counterModel from "./models/counter"
import studentsModel from "./models/students"

//得到一个dva对象
const app = dva();

//在启动之前定义模型
app.model(counterModel)
app.model(studentsModel)

//设置根路由，即启动后，要运行的函数，函数的返回结果会被渲染
app.router(() => <App />)

app.start("#root")
```

## counter.js组件

```javascript
import React, { useRef } from 'react'
import { connect } from "dva"

function Counter(props) {
    const inp = useRef();
    return (
        <div>
            <h1>{props.number}</h1>
            <button onClick={props.onAsyncDecrease}>异步减</button>
            <button onClick={props.onDecrease}> - </button>
            <button onClick={props.onIncrease}> + </button>
            <button onClick={props.onAsyncIncrease}>异步加</button>
            <p>
                <input type="number" ref={inp} />
                <button onClick={() => {
                    const n = parseInt(inp.current.value);
                    props.onAdd(n);
                }}>加上</button>
            </p>
        </div>
    )
}

const mapStateToProps = state => ({
    number: state.counter
})

const mapDispatchToProps = dispatch => ({
    onIncrease() {
        dispatch({
            type: "counter/increase"
        })
    },
    onDecrease() {
        dispatch({
            type: "counter/decrease"
        })
    },
    onAdd(n) {
        dispatch({ type: "counter/add", payload: n })
    },
    onAsyncIncrease() {
        dispatch({ type: "counter/asyncIncrease" })
    },
    onAsyncDecrease() {
        dispatch({ type: "counter/asyncDecrease" })
    }
})

export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```

## model文件夹counter.js

```javascript
export default {
    namespace: "counter",
    state: 0,
    reducers: {
        increase(state) {
            return state + 1;
        },
        decrease(state) {
            return state - 1;
        },
        add(state, { payload }) {
            return state + payload;
        }
    },
    effects: {
        *asyncIncrease(action, { call, put }) {
            yield call(delay, 1000);
            yield put({ type: "increase" })
        },
        *asyncDecrease(action, { call, put }) {
            yield call(delay, 1000);
            yield put({ type: "decrease" })
        }
    },
    subscriptions: {
        resizeIncrease({ dispatch }) {
            //订阅窗口尺寸变化，每次变化让数字增加
            window.onresize = () => {
                dispatch({ type: "increase" })
            }
        },
        resizeDecrease({ dispatch, history }) {
            history.listen(() => {
                dispatch({ type: "decrease" })
            })
        }
    }
}

function delay(duration) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve()
        }, duration);
    })
}
```
