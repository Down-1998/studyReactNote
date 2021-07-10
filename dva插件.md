# dva插件

通过```dva对象.use(插件)```，来使用插件，插件本质上就是一个对象，该对象与配置对象相同，dva会在启动时，将传递的插件对象混合到配置中。

## dva-loading

该插件会在仓库中加入一个状态，名称为loading，它是一个对象，其中有以下属性

- global：全局是否正在处理副作用（加载），只要有任何一个模型在处理副作用，则该属性为true
- models：一个对象，对象中的属性名以及属性的值，表示哪个对应的模型是否在处理副作用中（加载中）
- effects：一个对象，对象中的属性名以及属性的值，表示是哪个action触发了副作用
**实例,点击按钮对state仓库中的counter属性进行加、减操作**

## 全局indexjs文件引入dva-loading插件

```javascript
import routerConfig from "./routerConfig"
import dva from "dva";
import counterModel from "./models/counter"
import studentsModel from "./models/students"
import { createBrowserHistory } from "history"
import createLoading from "dva-loading"

//得到一个dva对象
const app = dva({
    history: createBrowserHistory()
});

app.use(createLoading()); //使用dva-loading插件

//在启动之前定义模型
app.model(counterModel)
app.model(studentsModel)

//设置根路由，即启动后，要运行的函数，函数的返回结果会被渲染
app.router(routerConfig)

app.start("#root")
```

counter组件,点击相应操作并于dva-loading结合
```javascript
import React, { useRef } from 'react'
import { connect } from "dva"
import Modal from "./components/common/Modal"

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
            {
                props.isLoading && <Modal>
                    <div style={{ color: "#fff", fontSize: "2em" }}>
                        加载中....
                    </div>
                </Modal>
            }
        </div>
    )
}

const mapStateToProps = state => ({
    number: state.counter,
    isLoading: state.loading.models.counter
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
