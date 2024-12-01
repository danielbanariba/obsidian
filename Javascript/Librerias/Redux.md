---
tags:
  - Redux
---
Con la poca información que tengo xd funciona igual que un Switch
Otro dato interesenta, que Redux, esta muy asociacido con aplicaciones con [[React]], pero igual se pueden utilizar en otros frenkwords



---
## Store



---
## Actions



---
## Reducers



 
---
## Instalación

Instalar Redux:
```shell
npm i @reduxjs/toolkit
```

```shell
npm i react-redux
```

```shell
yarn add @reduxjs/toolkit
```



---
## Importamos la libreria

```js
// Importing 
import { configureStore } from '@reduxjs/toolkit';// Creates a store
import { createSlice } from '@reduxjs/toolkit'; // Combines and creates reducer
```



---
## Funciones

Las funciones mas importantes y las mas utilizadas son 
- **configureStore**: Envuelve la función createStore del Redux básico. Proporciona automáticamente middlewares y enhancers.
- **createReducer**: Reemplaza la implementación clásica del reducer, haciendo el código más corto y simple de entender.
- **createAction()**: Una utilidad que retorna una función creadora de acciones.
- **createSlice()**: Una función que convenientemente reemplaza tanto createAction como createReducer con una única función.
- **createAsyncThunk()**: Toma cadenas de Redux como argumentos y retorna una Promesa.
- **createEntityAdapter()**: Una utilidad que ayuda a realizar operaciones CRUD.\

#### Ejemplos de codigo de cada funcion
```js
// 1. configureStore
import { configureStore } from '@reduxjs/toolkit'
import rootReducer from './reducers'

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(logger),
  devTools: process.env.NODE_ENV !== 'production'
})

// 2. createReducer
import { createReducer } from '@reduxjs/toolkit'

const initialState = { value: 0 }

const counterReducer = createReducer(initialState, (builder) => {
  builder
    .addCase('INCREMENT', (state) => {
      state.value += 1
    })
    .addCase('DECREMENT', (state) => {
      state.value -= 1
    })
    .addCase('RESET', (state) => {
      state.value = 0
    })
})

// 3. createAction
import { createAction } from '@reduxjs/toolkit'

const increment = createAction('INCREMENT')
const decrement = createAction('DECREMENT')
const reset = createAction('RESET')

// Uso:
console.log(increment())      // { type: 'INCREMENT' }
console.log(increment(5))     // { type: 'INCREMENT', payload: 5 }

// 4. createSlice
import { createSlice } from '@reduxjs/toolkit'

const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      state.push({ id: Date.now(), text: action.payload, completed: false })
    },
    toggleTodo: (state, action) => {
      const todo = state.find(todo => todo.id === action.payload)
      if (todo) {
        todo.completed = !todo.completed
      }
    },
    removeTodo: (state, action) => {
      return state.filter(todo => todo.id !== action.payload)
    }
  }
})

export const { addTodo, toggleTodo, removeTodo } = todosSlice.actions
export default todosSlice.reducer

// 5. createAsyncThunk
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit'

const fetchUserById = createAsyncThunk(
  'users/fetchById',
  async (userId, thunkAPI) => {
    const response = await fetch(`https://api.example.com/users/${userId}`)
    return response.json()
  }
)

const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    loading: 'idle',
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUserById.pending, (state) => {
        state.loading = 'loading'
      })
      .addCase(fetchUserById.fulfilled, (state, action) => {
        state.loading = 'idle'
        state.data = action.payload
      })
      .addCase(fetchUserById.rejected, (state, action) => {
        state.loading = 'idle'
        state.error = action.error
      })
  }
})

// 6. createEntityAdapter
import { createEntityAdapter, createSlice } from '@reduxjs/toolkit'

const todosAdapter = createEntityAdapter({
  // Asume que cada todo tiene un campo `id`
  sortComparer: (a, b) => a.createdAt.localeCompare(b.createdAt)
})

// El adaptador proporciona funciones como `addOne`, `addMany`, `updateOne`, etc.
const todosSlice = createSlice({
  name: 'todos',
  initialState: todosAdapter.getInitialState({
    loading: 'idle'
  }),
  reducers: {
    todoAdded: todosAdapter.addOne,
    todosReceived: todosAdapter.setAll,
    todoUpdated: todosAdapter.updateOne
  }
})

// Los selectores se generan automáticamente
const {
  selectAll: selectAllTodos,
  selectById: selectTodoById,
  selectIds: selectTodoIds
} = todosAdapter.getSelectors((state) => state.todos)
```



---
