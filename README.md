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
