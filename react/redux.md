### createStore
```js
export function createStore(reducer) {
  let state = null;
  const listeners = [];
  const subscribe = listener => listeners.push(listener);
  const getState = () => state;
  const dispatch = action => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  };
  dispatch({});
  return { getState, dispatch, subscribe };
}

```
```js
let initState = {
  name: "若邪",
  description: "666666"
};
export default function(state, action) {
  if (!state) {
    state = initState;
  }
  switch (action.type) {
    case "SET_NAME":
      return {
        ...state,
        name: action.name
      };
    case "SET_DESCRIPTION":
      return {
        ...state,
        description: action.description
      };
    default:
      return state;
  }
}
```
```js
import { createStore } from "./createStore";

import reducer from "./reducer";

let store = createStore(reducer);

store.subscribe(() => {
  let state = store.getState();
  console.log(state.name, state.description);
});
setInterval(() => {
  store.dispatch({
    type: "SET_DESCRIPTION",
    description: new Date()
  });
}, 2000);
```
### combineReducers
```js
export default function combineReducers(reducers) {

  const reducerKeys = Object.keys(reducers)

  /*返回合并后的新的reducer函数*/
  return function combination(state = {}, action) {
    /*生成的新的state*/
    const nextState = {}

    /*遍历执行所有的reducers，整合成为一个新的state*/
    for (let i = 0; i < reducerKeys.length; i++) {
      const key = reducerKeys[i]
      const reducer = reducers[key]
      /*之前的 key 的 state*/
      const previousStateForKey = state[key]
      /*执行 分 reducer，获得新的state*/
      const nextStateForKey = reducer(previousStateForKey, action)

      nextState[key] = nextStateForKey
    }
    return nextState;
  }
}
```

### applyMiddleware


