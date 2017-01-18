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

### Example 5 - Normalizing state shape

```javascript
const todosById = (state = {}, action) => {
    switch (action.type) {
        case 'ADD_TODO':
        case 'TOGGLE_TODO':
            return {
                ...state,
                [action.id]: todo(state[action.id], action)
            };
        default:
            return state;
    }
};

const todosAllIds = (state = [], action) => {
    switch (action.type) {
        case 'ADD_TODO':
            return [...state, action.id];
        default:
            return state;
    }
};

const todos = combineReducers({ todosById, todosAllIds });
```

### Example 6 - Loading state

```javascript
const isFetching = (state = false, action) => {
    if (action.filter !== filter) {
        return state;
    }

    switch(action.type) {
        case 'REQUEST_TODOS':
            return true;
        case 'RECEIVE_TODOS':
            return false;
        default:
            return state;
    }
};
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

### Example 2 - Initial state

***Obs*** *Dan Abramov does not recommend*

```javascript
const persistedState = {
    todos: [{
        id: '0',
        text: 'Welcome back!',
        completed: true
    }]
};
const store = createStore(reducers, persistedState);
```

### Example 3 - redux-logger implementation

```javascript
const addLoggingToDispatch = (store) => {
    const next = store.dispatch;
    if (!console.group) {
        return next;
    }

    return (action) => {
        console.group(action.type);
        console.log('%c prev state', 'color: gray', store.getState());
        console.log('%c action', 'color: blue', action);
        const returnValue = next(action);
        console.log('%c next state', 'color: green', store.getState());
        console.groupEnd(action.type);

        return returnValue;
    }
}

const store = createStore(reducers);
if (process.env.NODE_ENV !== 'production') {
    store.dispatch = addLoggingToDispatch(store);
}
```

## Middleware

### Simple implementation

```javascript
const applyMiddlware = (store, middlewares) => {
    middlewares.slice().reverse().forEach(middleware =>
        store.dispatch = middleware(store)(store.dispatch);
    );
};
```

### Example

```javascript
const store = createStore(reducers, applyMiddlware(middleware1, middlware2));
```

## Promise as action (redux-promise)

### Simple implementation

```javascript
const promise = (store) => (next) => (action) => {
    if (typeof action.then === 'function') {
        return action.then(next);
    }
    return next(action);
};
```


### Example

```javascript
const store = createStore(reducers, [promise]);

const receiveTodos = (filter, response) => ({
    type: 'RECEIVE_TODOS',
    filter,
    response
});

const fetchTodos = (filter) => {
    return api.fetchTodos(filter).then(response =>
        receiveTodos(filter, response)
    );
};

const fetchData = () => {
    const { filter, fetchTodos, requestTodos } = this.props;
    requestTodos(filter);
};
```

## Redux Thunk (redux-thunk)

### Simple implementation

```javascript
const thunk = (store) => (next) => (action) => {
    if (typeof action === 'function') {
        action(store.dispatch, store.getState);
    } else {
        next(action);
    }
}
```

### Example 1 - Before

```javascript
const receiveTodos = (filter, response) => ({
    type: 'RECEIVE_TODOS',
    filter,
    response
});

const fetchTodos = (filter) => {
    return api.fetchTodos(filter).then(response =>
        receiveTodos(filter, response)
    );
}

const requestTodos = (filter) => ({
    type: 'REQUEST_TODOS',
    filter
});

const fetchData = () => {
    const { filter, fetchTodos, requestTodos } = this.props;
    requestTodos(filter);
    fetchTodos(filter);
};
```

### Example 1 - After

```javascript
const receiveTodos = (filter, response) => ({
    type: 'RECEIVE_TODOS',
    filter,
    response
});

const fetchTodos = (filter) => (dispatch, getState) => {
    if (getIsFetching(getState(), filter)) {
        return Promise.resolve();
    }

    dispatch(requestTodos(filter));

    return api.fetchTodos(filter).then(response => {
        dispatch(receiveTodos(filter, response));
    });
}

const requestTodos = (filter) => ({
    type: 'REQUEST_TODOS',
    filter
});

const fetchData = () => {
    const { filter, fetchTodos } = this.props;
    fetchTodos(filter).then(() => console.log('done!'));
};
```

### Example 2

```javascript
const errorMessage = (state = null, action) => {
    if (filter !== action.filter) {
        return state;
    }

    switch (action.type) {
        case 'FETCH_TODOS_FAILURE':
            return action.message;
        case 'FETCH_TODOS_REQUEST':
        case 'FETCH_TODOS_SUCCESS':
            return null;
        default:
            return state;
    }
}

const fetchTodos = (filter) => (dispatch, getState) => {
    if (getIsFetching(getState(), filter)) {
        return Promise.resolve();
    }

    dispatch({
        type: 'FETCH_TODOS_REQUEST',
        filter
    });

    return api.fetchTodos(filter).then(response => {
        dispatch({
            type: 'FETCH_TODOS_SUCCESS',
            filter,
            response
        });
    },
    error => {
        dispatch({
            type: 'FETCH_TODOS_FAILURE',
            filter,
            message: error.message || 'Something went wrong'
        });
    });

    // Do not use .catch() cause it can catch error from the dispatch too
}
```

## Filters

### Example

```javascript
const getIds = (state) => state.ids;
const getIsFetching = (state) => state.isFetching;

const getVisibleTodos = (state, filter) => {
    const ids = getIds(state.listByFilter[filter]);
    return ids.map(id => fromById.getTodo(state.todosById, id));
};

const getIsFetching = (state, filter) => {
    return getIsFetching(state.listByFilter[filter]);
}
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

## Action creators

### Example

```javascript
let nextTodoId = 0;

const addTodo = (text) => ({
    type: 'ADD_TODO',
    id: nextTodoId++,
    text
});

store.dispatch(addTodo("Novo todo"));
```





## React integration

### Subscribe and Unsubscribe

```javascript
componentDidMount() {
    const { store } = this.props;
    this.unsubscribe = store.subscribe(() => {
        this.forceUpdate();
    });
}

componentWillUnmount() {
    this.unsubscribe();
}
```

### Store provider

***Obs*** *I don't recomend doing this. I prefer to pass the store via props to every single component. Explicit is better than implicit.*

#### Simple implementation

```javascript
class Provider extends Component {
    getChildContext() {
        return {
            store: this.props.store
        };
    }

    render() {
        return this.props.children;
    }
}
Provider.childContextTypes = {
    store: React.PropTypes.object
};
```

#### Example

```javascript
import { Provider } from 'react-redux';

ReactDOM.render(
    <Provider store={createStore(todoApp)}>
        <TodoApp />
    </Provider>,
    document.getElementById('root')
);

class VisibleTodoList extends Component {
    componentDidMount() {
        const { store } = this.context;
        this.unsubscribe = store.subscribe(() => {
            this.forceUpdate();
        });
    }

    componentWillUnmount() {
        this.unsubscribe();
    }

    render() {
        const props = this.props;
        const { store } = this.context
        const state = store.getState();

        return (
            <TodoList
                todos={getVisibleTodos(state.todos, state.visibilityFilter)}
                onTodoClick={id => store.dispatch({
                    type: 'TOGGLE_TODO',
                    id
                })}
            />
        )
    }
}
VisibleTodoList.contextTypes = {
    store: React.PropTypes.object
};

const AddTodo = (props, { store }) => {
    let input;

    return (
        <div>
            <input ref={node => { input = node }} />
            <button onClick={() => {
                store.dispatch({
                    type: 'ADD_TODO',
                    id: nextTodoId++,
                    text: input.value
                })
                input.value = '';
            }}>
                Add Todo
            </button>
        </div>
    );
};
AddTodo.contextTypes = {
    store: React.PropTypes.object
};
```

### Using react-redux connect

#### Example 1 - ChildOne converted to react-redux connect

```javascript
import { connect } from 'react-redux';

const mapStateToProps = (state) => {
    return {
        todos: getVisibleTodos( state.todos, state.visibilityFilter )
    };
}

const mapDispatchToProps = (dispatch) => {
    return {
        onTodoClick: (id) => dispatch({ type: 'TOGGLE_TODO', id })
    };
}

const ChildOne = connect( mapStateToProps, mapDispatchToProps )(TodoList);
```

#### Example 2 - Generating containers

```javascript
let AddTodo = ({ dispatch }) => {
    let input;

    return (
        <div>
            <input ref={node => { input = node }} />
            <button onClick={() => {
                dispatch({
                    type: 'ADD_TODO',
                    id: nextTodoId++,
                    text: input.value
                })
                input.value = '';
            }}>
                Add Todo
            </button>
        </div>
    );
};

// This won't be used since no states are needed
// So it will be passed null to connect and therefore
// avoid subscription to the store
const mapStateToProps = (state) => {
    return {};
};

// This is the default function if we don't specify any parameter for
// mapDispatchToProps
const mapDispatchToProps = (dispatch) => {
    return { dispatch };
}

// connect(mapStateToProps, mapDispatchToProps) is almost the same thing
// although if mapStateToProps is a falsy value it prevents the component to
// subscribe to the store.
// Same as AddTodo = connect(null, mapDispatchToProps);
AddTodo = connect()(AddTodo);
```

#### Example 3 - Receiving props as second argument

```javascript
const mapStateToProps = (state, ownProps) => {
    return {
        active: ownProps.filter === state.visibilityFilter
    }
};

const mapDispatchToProps = (dispatch, ownProps) => {
    onClick: () => {
        dispatch({
            type: 'SET_VISIBILITY_FILTER',
            filter: ownProps.filter
        });
    }
};

const FilterLink = connect(mapStateToProps, mapDispatchToProps)(Link);
```
