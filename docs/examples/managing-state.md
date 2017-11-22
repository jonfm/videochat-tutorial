# Managing state

## Background

Hold on to our tokens and user information.

## React (Redux)

### Store

```js
import { createStore, compose, combineReducers } from 'redux'

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





## Vue

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

