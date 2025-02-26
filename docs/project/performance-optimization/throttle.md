# react跨组件状态共享


## 应用场景
在项目开发初期,我们在字典字段的筛选和展示组件中调用了字典查询接口,随着业务的发展，出现了一张页面中存在多个同类字典的展示，这样会
导致字典接口的重复调用,在不修改加载流程的前提下解决接口重复调用问题，便引入了跨组件状态共享的机制，来优化相同接口频繁调用问题。


## 基础原理

使用useContext进行全局状态存储，使用useReducer进行状态管理来实现跨组件状态共享。

- useContext：用于在组件树中共享状态，避免通过多层组件传递 props，从而简化组件之间的数据传递。

- useReducer：用于管理复杂的状态逻辑，尤其是当状态逻辑涉及多个子值或下一个状态依赖于前一个状态时。它类似于 Redux 的 reducer 函数。

通过结合使用 useContext 和 useReducer，可以实现一个轻量级的状态管理方案，类似于 Redux，但完全基于 React 的原生 API。


## 基础实现

```jsx
import React, { createContext, useContext, useEffect, useReducer, useState } from 'react';

const initialCache = {};

const regCache = (state, action) => {
  let result = { ...state };
  if (!result[action.payload.key]) {
    result[action.payload.key] = action.payload;
  }
  return result;
};

function reducer(state, action) {
  switch (action.type) {
    case 'regRequest':
      return regCache(state, action);
    case 'UPDATE_STATE':
      return action.payload;
    default:
      return state;
  }
}

const Context = createContext({});

function useThrottle() {
  return useContext(Context);
}

function ThrottleProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialCache);
  const [cachePool, setCachePool] = useState({});

  const regRequest = (key, fun) => {
    dispatch({
      type: 'regRequest',
      payload: {
        key,
        fn: fun,
      },
    });
  };

  useEffect(async () => {
    let current = { ...state };
    let reqCount = 0;
    for (let key in current) {
      if (!current[key].data && !current[key].status) {
        reqCount++;
        current[key].status = 'padding';
        dispatch({ type: 'UPDATE_STATE', payload: current });
        current[key].data = await current[key].fn(current[key].param || key);
      }
    }
    if (reqCount !== 0) {
      dispatch({ type: 'UPDATE_STATE', payload: current });
    }
  }, [state]);

  const getResponse = (key) => {
    return state[key]?.data;
  };

  return <Context.Provider value={[state, regRequest, getResponse]}>{children}</Context.Provider>;
}

export { useGlobalState, GlobalStateProvider };

```
提供了provider组件和hook，在组件中使用useGlobalState的state，regRequest，getResponse来实现共享状态的注册,监控,获取。

## 使用说明

### 引入全局状态组件
```jsx
import { GlobalStateProvider } from '@/components/base/global-state-provider';

const appConfig = {
  app: {
    rootId: 'root',
    addProvider: ({children}) => {
      return (
              <GlobalStateProvider>

              </GlobalStateProvider>
      );
    },
    ...
  }

};

runApp(appConfig);

```
### 组件使用共享状态

```js
import { useGlobalState } from '@/components/base/global-state-provider';

const myComponent = (props) => {


  const [result, regRequest, getResponse] = useGlobalState();

  useEffect(async () => {
      
      ... //省略代码
    
      regRequest(dicCode, DicsService.getDics);
    
      ...
    
  }, []);

  useEffect(() => {
    let result = getResponse(dicCode);

    if(result){
    ...
    }

  }, [result]);

  ...

};


```

### 组件说明

#### useGlobalState

- state: 全局状态对象
- regRequest: 注册全局状态,参数说明：
  - key:  节流标识
  - fn:  节流的方法
  - param： 节流传参，默认值为节流标识，非必传
- getResponse: 获取全局状态,参数说明
  - key: 节流标识



