# [React-imvc、Remesh、Pure-model的状态管理方式](https://github.com/silver-blinder/GitBlog/issues/4)

# **React-imvc：**

顾名思义，MVC架构+ isomorphic（同构，即只编写一份代码，在 NodeJS 里做服务端渲染（SSR），在浏览器里做客户端渲染。

### 数据的更新与获取（数据的流动）：

state整体分为两类：initial的以及后来update的：

### initial：

1. Controller中暴露的方法**getInitialState（根据SSR flag的设定 可在服务端/客户端）**
    
    getInitialState的入参 initialState 可以直接定义 也可以使用model中的定义（优先级更高，源码如下）：
    
    ```
    // imvc源码：
    // 如果 Model 存在，且 initialState 和 actions 不存在，从 Model 里解构出来*
    // 当同时使用 Model 和 initialState 属性时，以 Model 的 initialState 为准。
    if (Model && initialState === undefined && actions === undefined) {
    let { initialState: $initialState, ...$actions } = Model
    	initialState = this.initialState = $initialState
    	actions = this.actions = $actions
    }
    ```
    
- 服务端执行**getInitialState时**：

```
服务端执行：
1. getInitialState() - 在服务端执行，获取初始状态
2. shouldComponentCreate() - 服务端判断是否需要创建组件
3. componentWillCreate() - 服务端组件创建前的逻辑
4. render() - 生成HTML字符串
5. 将状态序列化到 window.__INITIAL_STATE__ 或类似的全局变量中（SEO友好的原因）

.....

// 应用启动时 通过下述方式读取（源码）
if (typeof __INITIAL_STATE__ !== 'undefined') {
  globalInitialState = __INITIAL_STATE__  // 获取服务端数据
  __INITIAL_STATE__ = void 0              // 清理掉，避免内存泄漏
}

.....

客户端执行：
1. 读取服务端序列化的状态 (globalInitialState)
2. 跳过 getInitialState()
3. 跳过 shouldComponentCreate() 
4. 跳过 componentWillCreate()
5. 直接进行 hydration（水合）
6. componentDidFirstMount() / componentDidMount()等 - 客户端生命周期

.....

组件内获取：
const state = useModelState()  // 拿到的state中initial的部分为服务端渲染
```

- 客户端执行**getInitialState时**：

```tsx
// 完整的客户端生命周期
1. getInitialState() - 在客户端执行
2. shouldComponentCreate() - 跳过
3. componentWillCreate() - 跳过
【关闭 SSR 后，不执行 componentWillCreate 和 shouldComponentCreate，
直接返回 Loading 界面（View属性）】
4. render() - 客户端渲染
5. componentDidFirstMount()(获取非首屏数据) / componentDidMount()(页面mount后浏览器
里相关的活动）等 - 客户端生命周期

// 组件内获取
const state = useModelState()  // 拿到的state中initial的部分为客户端渲染
```

### update：

通过Model Action Update state：

例如：

```jsx
export const UPDATE_STATE: Action<State, Partial<State>> = 
(state, payload: Partial<State>) => {
  return {
    ...state,
    ...payload,
  }
}

....

const { UPDATE_STATE } = this.store.actions
UPDATE_STATE({
	somestate,
	....
})

// 组件内获取
const state = useModelState()  // 可以拿到update的state

```

# Remesh：

以domain为核心的数据管理框架：

state相关（useState+useContext）：
state：定义数据结构及default值（相当于schema，不可读）

query：暴露给组件的query方法，获取state的值

command：修改state的方法（支持数据的部分更新；domin内直接调用，组件内用send进行数据更新）

effect相关（useEffect）

event：和useEffect的依赖项trigger类似，暴露一个event trigger，在拿到关键数据（合适的逻辑点）后在组件内执行响应操作。

api相关：
extern：统一管理api接口；getExtern统一获取。

总：

优点是确实可以做到数据层的隔离和统一管理：

不同类型的信息state（及对应的query、command）可以统一包装管理；

Task的方法可以简洁的处理由异步接口而来的数据，最终通过onSuccess统一进行command操作进行数据更新；

缺点是维护过程过于繁琐，实际开发中无法快速定位到数据的set是在哪一部分

所以我觉得对于类似于info、detail之类数据结构复杂、数据来源统一（例如来自于特定的一/两个接口）、高复用（在多个组件内都要用到）的数据确实更清晰，但对于例如show flag / 低复用性（只在特定组件内用到）的数据而言，useState可能是更好的解决方案。

# Pure-model：

MOP架构，实际上我觉得和react-imvc很像，只不过将controller的一部分功能移到了model里，单独定义并通过暴露的hooks在view层直接使用，主要解决跨端复用的问题。

关键方法：
setupStore：定义state和对应的reducer（action）

createReactModel：将用来定义 storesetupStore的Model Hooks转换成 React-Hooks 的 Model.useState（View层调用）

setupPreloadCallback —> SSR （基本同Controller 的 getInitialState）

setupStartCallback  （基本同Controller 的 componentDidMount）

ctx（setupContext createModelContext）：上下文对象，联通Model 和 View（controller）层的数据传输。

好处是将堆积在 View 层和 Controller 层的部分代码实现，放到了 Model 层维护，在 View 层和 Controller 层只留下函数调用的少量代码。同样带来的我觉得是繁琐，需要维护由pure model定义的Model.useState的state 和 可能由imvc本身 useModelState 的state

---

只是项目过程中的一点点理解，有机会会不断更新修改。