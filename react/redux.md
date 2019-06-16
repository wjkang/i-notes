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
### 
