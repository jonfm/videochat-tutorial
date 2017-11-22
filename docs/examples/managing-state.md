# Managing state

## Background

Hold on to our tokens and user information.

## React (Redux)

### Store & Reducer(s)

```js
import { createStore, compose, combineReducers } from 'redux'
import adapter from 'redux-localstorage/lib/adapters/localStorage'

// Specific reducer
const initialState = {
  user: null
}

function userReducer (state = initialState, action) {
  switch (action.type) {
    case SET_USER:
      return set(state, action.user) // merge helper
      default:
      return state
  }
}

// Combining reducers
const reducer = compose(
  mergePersistedState()
)(combineReducers({
  user:           userReducer,
  status:         statusReducer
}))

// Add localstorage
const storage = compose()(adapter(window.localStorage))

const enhancer = compose(
  persistState(storage, STORAGE_KEY)
)

// combine our store with our persistence strategy
export const store = createStore(reducer, enhancer)

```



### Component or Router

```js
import { Provider } from 'react-redux'

// Inject the store via props
ReactDOM.render(
  <Provider store={store}>
    <Router> ... </Router>
  </Provider>
)

// Optionaly define some action creators
export function setUser (user, jwt) {
    return { type: SET_USER, user, jwt }
}

// Dispatch the action where necessary
store.dispatch(setUser(user, jwt))

```

- Create the store
- Create reducers and action and combine them as required
- Pass the store as props

## Vue

Disclaimer: You can actually use redux with vue and it behaves as the above `vue-redux`

### Store

```js
export const store = new Vuex.Store({
  plugins: [
    createPersist({ // Persist our state in localstorage
      namespace: 'n3'
    })
  ],
  strict: true, // not for production
  state: {
    user: {
      jwt: undefined
    }
  },
  mutations: {
    addJwt (state, jwt) {
      state.user.jwt = jwt
    }
  },
  getters: {
    isLoggedIn: (state) => {
      return !!state.user.jwt
    }
  }
})

// Actions
export function login (username, password) {
  return auth(username, password).then(function (jwt) {
    store.commit('addJwt', jwt)
  })
}
```



### Router

```js
const ensureLoggedIn = (to, from, next) => {
  if (!store.getters.isLoggedIn) {
    return next('/login')
  } else {
    return next()
  }
}
```



### Component

```js
// main.js
new Vue({
  el: '#app',
  router,
  store, // injects the store into
  template: '<App/>',
  components: { App }
})

// Componend.vue
this.$store.getters.isLoggedIn
```



- Use `vuex` and `vuex-localstorae`
- explicitly defined `getters`which can compute values
- mutations are handled through `store.commit`

