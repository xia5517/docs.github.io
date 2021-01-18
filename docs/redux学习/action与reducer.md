# redux

> author: baoying
## data flow

![图片](https://agroup-bos-bj.cdn.bcebos.com/bj-d5c8209824d1946c02ce81723e35eae54b00fd1a)

### 一、调用 `store.dispatch(action)`

action 是一个描述发生的事情的简单对象。例如：`{ type: 'LIKE_ARTICLE', articleId: 42 } ` 
可以将 action 看作是一个指令，可以在应用程序中的任何位置调用 store.dispatch(action)。


### 二、调用 reducer

Store 将向 reducer 传递两个参数：当前状态树和操作。

```
// The current application state (list of todos and chosen filter)  
let previousState = [       
	'Read the docs.',        
]   
// The action being performed (adding a todo)  
let action = {   
	type: 'ADD_TODO', 
	text: '添加的todo'  
}   

// Your reducer returns the next application state  
let nextState = todoApp(previousState, action) 


// todoApp 是自己写的 reducer
const todoApp = (state = [], action) => {
    switch (action.type) {
        case 'ADD_TODO':
            return [
                ...state,
                action.text
            ]
        default:
            return state
    }
  }
```
reducer 是一个纯函数。它只计算下一个状态。它应该是完全可预测的：多次调用相同的输入应该产生相同的输出。它不应该执行API调用或路由器转换等任何副作用。这些应该在动作发出之前发生。

### 三、reducer root 将多个 reducer 组合成状态树
通过Redux 附带一个 `combineReducers` 辅助函数 ，将多个分支的 reducer 组合，每个分支管理状态树的一部分。
```
const todos = (state = [], action) => {
    switch (action.type) {
        case 'ADD_TODO':
            return [
                ...state,
                text
            ]
        default:
            return state
    }
  }

const visibilityFilter = (state = 'SHOW_ALL', action) => {
    switch (action.type) {
      case 'SET_VISIBILITY_FILTER':
        return action.filter
      default:
        return state
    }
  }
let rootReducer = combineReducers({    todos,    visibleTodoFilter  }) 

const store = createStore(rootReducer);

```

### 四、redux 存储 reducer root 组合好的完整状态树

完整状态树：
```
{
	todos: ['Read the docs.', '添加的todo'],
	visibleTodoFilter: 'SHOW_ALL'
}
```

现在这棵新状态树是你的应用程序的next state。会被每个注册 store.subscribe(listener) 的监听者收到新状态；听众可能会调用store.getState()来获取当前状态。现在，UI 可以更新以反映新的状态。


### 五、UI 更新

以上是 redux 的数据流，数据变化了，但是需要手动调用react 的render 方法，才能将数据变化展现在页面上。
   

 1. `subscribe(listener)`发布订阅模式实现自动渲染。
 2. react-redux 的connect 可以实现 store 的改变触发render 重新渲染。


## action 与 actions creater

Action 是改变 store 的唯一数据源，通过 store.dispatch() 将 action 传到 store。

action是个javascript 对象：
```
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

dispach 接受的是 action, 以添加一个todo项为例子，view 中的写法可能是：

```
	
	 <form
        onSubmit={e => {
          e.preventDefault()
          if (!input.value.trim()) {
            return
          }
          dispatch({type: 'ADD_TODO', text: input.value, id: global.id++})
          input.value = ''
        }}
      >
        <input ref={node => (input = node)} />
        <button type="submit">Add Todo</button>
      </form>
```

`dispatch({type: 'ADD_TODO', text: input.value, id: global.id++}) ` 每次生成一个action，将此逻辑抽离为通用逻辑就是 actions creater：
```
let nextTodoId = 0
export const addTodo = text => ({
  type: 'ADD_TODO',
  id: nextTodoId++,
  text
})
```

view:
```
<form
        onSubmit={e => {
          e.preventDefault()
          if (!input.value.trim()) {
            return
          }
          dispatch(addTodo(input.value))
          input.value = ''
        }}
      >
```

## reducer 将应用状态响应到store

 actions 只是描述了有事情发生了这一事实，并没有描述应用如何更新 state，reducer 则是真正改变 store 的过程。


```

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

```


### 问题&&原因

#### 为什么 reducer 是纯函数？
《javascript 函数式编程》中对纯函数的特点解释：
- 其结果只能从它的参数的值来计算
- 不能依赖于能被外部操作改变的数据
- 不能改变外部的状态
                   
简单来说，一个**函数返回的结果只依赖于它的参数**，在**执行过程没有副作用**就是纯函数。

1、 函数返回的结果只依赖于它的参数

```
const a = 1
const foo = b => a + b
foo(5)
```
foo函数的返回值依赖于外部变量a，a不是函数的参数，可能会被不知情的情况下被更改，造成foo的调用返回值不可预料，因此foo 不是纯函数

```
const foo = (a, b) => a + b
foo(5, 10)
```

 函数返回的结果只依赖于它的参数，可以保证相同的输入，不论何时何地，返回值相同。
 
2、执行过程没有副作用
```
const calu = (obj, num) => {
	obj.money = 2;
	return obj.x + num;
}
const person = {money: 5}
calu(person, 10)
```
calu 函数对外部的 person 造成了修改，产生了副作用，篡改了外部的对象，因此它也不是和纯函数。

那么为什么要用 reducer 为什么要用 纯函数？


https://github.com/reduxjs/redux/blob/master/src/combineReducers.ts

![图片](https://agroup-bos-bj.cdn.bcebos.com/bj-18226597007b898f78c048dc3fe2e02f462922d1)


```
var nextStateForKey = reducer(previousStateForKey, action)
hasChanged = hasChanged || nextStateForKey !== previousStateForKey
```
拿到下next state 后要比较新旧 state 是否一致，比较的是两个对象的存储位置，也就是浅比较法，所以，当我们 reducer 直接返回旧的 state 对象时，Redux 认为没有任何改变，从而导致页面没有更新。