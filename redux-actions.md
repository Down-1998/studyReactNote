# redux-actions

> 该库用于简化action-types、action-creator以及reducer
> 官网文档：https://redux-actions.js.org/

## createAction(s)

### createAction

该函数用于帮助你创建一个action创建函数（action creator）

### createActions

该函数用于帮助你创建多个action创建函数

## handleAction(s)

### handleAction

简化针对单个action类型的reducer处理，当它匹配到对应的action类型后，会执行对应的函数

### handleActions

简化针对多个action类型的reducre处理

## combineActions

配合createActions和handleActions两个函数，用于处理多个action-type对应同一个reducer处理函数。

```javascript
import { createActions, handleActions, combineActions } from "redux-actions"

//创建actions,将type作为键,将payload作为第二个参数(函数)返回值
const actions = createActions({
    INCREASE: () => 1,
    DECREASE: () => -1,
    ASYNC_INCREASE: null,
    ASYNC_DECREASE: null,
    ADD: number => number
});

//actions是一个对象,会将action的扁平化对象,以小驼峰来命名
export const { increase, decrease, asyncIncrease, asyncDecrease, add } = actions

//简化reduer函数
export default handleActions({
    [combineActions(increase, decrease, add)]: (state, { payload }) => state + payload
}, 0)
```