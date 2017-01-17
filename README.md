# Redux

## The Reducer

***Obs:*** *The reducer function usually have the same name as the property in the store.*

### Example 1

```javascript
const counter = (state = 0, action) => {
    switch (action.type) {
        case 'INCREMENT':
            return state + 1;
        case 'DECREMENT':
            return state - 1;
        default:
            return state;
    }
}
```

### Example 2

```javascript
const todos = (state = [], action) => {
    switch (action.type) {
        case 'ADD_TODO':
            return [
                ...state,
                {
                    id: action.id,
                    text: action.text,
                    completed: false
                }
            ];
        case 'TOGGLE_TODO':
            return state.map(todo => {
                if (todo.id !== action.id) {
                    return todo;
                }

                return {
                    ...todo,
                    completed: !todo.completed
                };
            })
        case default:
            return state;
    }
};
```

### Example 3 - Reducer composition with arrays

```javascript
const todo = (state, action) => {
    switch (action.type) {
        case 'ADD_TODO':
            return {
                id: action.id,
                text: action.text,
                completed: false
            };
        case 'TOGGLE_TODO':
            if (todo.id !== action.id) {
                return state;
            }

            return {
                ...state,
                completed: !state.completed
            };
        default:
            return state;
    }
}

const todos = (state = [], action) => {
    switch (action.type) {
        case 'ADD_TODO':
            return [
                ...state,
                todo(undefined, action)
            ];
        case 'TOGGLE_TODO':
            return state.map(t => todo(t, action));
        case default:
            return state;
    }
};
```

### Example 4 - Reducer composition with objects

```javascript
const visibilityFilter = (state = 'SHOW_ALL', action) => {
    switch (action.type) {
        case 'SET_VISIBILITY_FILTER':
            return action.filter;
        default:
            return state;
    }
};

const todoApp = (state = {}, action) => {
    return {
        todos: todos( state.todos, action ),
        visibilityFilter: visibilityFilter( state.visibilityFilter, action )
    };
}
```

### Combine reducers

#### Simplified implementation

```javascript
const combineReducers = (reducers) => (state = {}, action) => {
    return Object.keys(reducers).reduce(
        (nextState, key) => {
            nextState[key] = reducers[key](state[key], action);
            return nextState;
        },
        {}
    );
};
```

#### Example

```javascript
import { combineReducers } from 'redux';
import { todos, visibilityFilter } from './reducers';

const todoApp = combineReducers({ todos, visibilityFilter });
```


## The Store

### Simplified implementation

```javascript
const createStore = (reducer) => {

    let state;
    let listeners = [];

    const getState = () => state;

    const dispatch = (action) => {
        state = reducer(state, action);
        listeners.forEach(listener => listener());
    };

    const subscribe = (listener) => {
        listeners.push(listener);

        return () => {
            listeners = listeners.filter(l => l !== listener);
        };
    };

    dispatch({});

    return { getState, dispatch, subscribe };

};
```

### Example 1

```javascript
import { createStore } from 'redux';

const counter = (state = 0, action) => {
    switch (action.type) {
        case 'INCREMENT':
            return state + 1;
        case 'DECREMENT':
            return state - 1;
        default:
            return state;
    }
}

const store = createStore(counter);
const render = () => {
    document.body.innerText = store.getState();
}
store.subscribe(render);
render();

document.addEventListener('mousedown', () => {
    store.dispatch({ type: 'INCREMENT' });
});

document.addEventListener('mouseup', () => {
    store.dispatch({ type: 'DECREMENT' });
});
```


## Avoiding array mutation

```javascript
const addCounter = (list, initialValue) => {
    return [...list, initialValue];
}

const removeCounter = (list, index) => {
    return [
        ...list.slice(0, index),
        ...list.slice(index + 1)
    ];
}

const incrementCounter = (list, index) => {
    return [
        ...list.slice(0, index),
        list[index] + 1,
        ...list.slice(index + 1)
    ];
}
```


## Avoiding object mutation

```javascript
const toggleTodo = (todo) => {
    return { ...todo, completed: !todo.completed };
}
```

## React integration

Currently watching: https://egghead.io/lessons/javascript-redux-extracting-container-components-filterlink?series=getting-started-with-redux
