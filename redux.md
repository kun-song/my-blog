# Redux

## 自述

`Redux` 是 JavaScript 的 **状态容器**，提供可预测的状态管理，使用 Redux 可以做到前端行为的 **完全可预测**，从而可以实现日志打印、热加载、时间旅行、同构应用、录制重放等功能，而无需任何开发参与。

#### 安装

需要三个库：

```
npm install --save redux

// React 绑定库
npm install --save react-redux

// 开发者工具
npm install --save-dev redux-devtools
```

#### 要点

Redux 中有个三个组件：`store`, `action`, `reducer`。

* `store` 存放状态树，保存所有 `state`，且在 Redux 中只有一个。
* `action` 是一个简单的 JavaScript 对象，**描述** 状态如何变化。
* `reducer` 是 **纯函数**，**实际执行** `action` 描述的状态变化。

## 三大原则

### 单一数据源

**整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中**。

### State 是只读的

**唯一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象**。

确保视图、网络请求都不能直接修改 state，他们只能 **表达修改的意图**，因为所有的修改都被集中化处理，且严格按照一个接一个的顺序执行，因此不用担心 race condition 的出现。

### Reducer 是纯函数

## 基础



