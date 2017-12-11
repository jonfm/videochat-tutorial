# Handling raw DOM elements

---

## Background

```js
// disable DOM manipulation from the 3rd party library
OT.initPublisher(null, { insertDefaultUI: false })
// grab the video element and pass it to the compenent
publisher.on('videoElementCreated', (videoElement) {
  // do something with videoElement
})
```

- Third party libs sometimes manipulate the DOM directly.
- With Tokbox above, luckily you can disable this and capture the elements directly.

---

## React

```jsx
componentDidMount() {
  if (this.tokboxContainer && this.props.video) {
    this.tokboxContainer.appendChild(this.props.video) 
  }
}

render() {
  return (
    <div ref={tokboxContainer => this.tokboxContainer = tokboxContainer }></div>
  )
}
```
- Use refs to insert the video into our virtual DOM
- Text-only refs are being deprecated in favour of the functional approach

---

## Vue 

```vue
<template>
  <div v-html="renderedPublisherVideo"></div>
</template>

<script>
export default {
  data() {
    return {
      props: ['publisherVideo'],
      computed: {
        renderedPublisherVideo: function () {
          return this.publisherVideo.innerHTML
        }
      }
    }
  }
}
</script>
```
- Use the `v-html` directive with a computed property

---

# Routing & Auth

---

## Background

Some routes are private. when these are accessed we need to ensure the user is logged in, otherwise redirect them to the login page. In turn, once they have logged in successfully, we want to redirect them to somewhere private (normally the place they tried to reach initially).

---

## React

### Router

(Old style)

```jsx
<Route path="/" component={Screen}>
  <IndexRoute onEnter={skipIfAuthorized} component={Auth}/>
  <Route path="profile" onEnter={authCheck} component={Profile}/>
</Route>
```

- Default is JSX, but you can map JS to JSX really easily
- onEnter hooks have been removed

---

- new example is a pure function, passing render as a prop

```jsx
<Route render={props => (
    props.authCheck ? (
      <Component {...props}/>
    ) : (
      <Redirect to={{ // or use the Login component directly
        pathname: '/login',
        state: { from: props.location }
      }}/>
    )
  )}
/>
```

---

### Component

```jsx
<form onSubmit={onLogin}>
  <input name="email" type="text" value={email} onChange={setValue}></input>
  <input name="password" type="password" value={email} onChange={setValue}></input>
</form>
```
```js
onLogin (evt) {
  evt.preventDefault()
  
  login( this.state.email, this.state.password ).then(user => {
     hashHistory.push('/')
  })
}
```
---

```js
setValue (evt) {
  const el = evt.target
  const name = el.getAttribute('name')

  this.setState({
    [name]: el.value
  })
}
```


- React deprecated the `valueLink` property in 2016
- We provide the `setValue` helper

---

## Vue

### Router


```js
export default new Router({
  routes: [
    {
      path: '/',
      name: 'Index',
      component: Index,
      beforeEnter: ensureLoggedIn
    },
    { path: '/login',
      name: 'Login',
      component: Login,
      props: { login, publisherVideo: publisherVideo },
      beforeEnter: skipIfLoggedIn
    }
  ]
})
```
---

```js
const ensureLoggedIn = (to, from, next) => {
  if (!store.getters.isLoggedIn) {
    return next('/login')
  } else {
    return next() // go to the next handler
  }
}
```

- Based on plain JS
- Limited to 1 beforeEnter hook so you have to chain these manually

---

### Component

```vue
<template>
  <form v-on:submit.prevent="onLogin">
    <input type="text" v-model.trim="username"/>
    <input type="password" v-model.trim="password"/>
  </form>
</template>
```

```js
methods: {
  onLogin: function () {
    this.login(this.username, this.password).then(() => {
      return this.$router.push('/')
    })
  }
}        
```

- The v-model directive handles the binding of changes to internal attributes for us out of the box which is nice.
- How does this work in practice?

---

# Managing state

---

## Background

Hold on to our tokens and user information.

---

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
```
---
```js
// Combining reducers
const reducer = compose(
  mergePersistedState()
)(combineReducers({
  user:           userReducer,
  status:         statusReducer
}))
```
```js
// Add localstorage
const storage = compose()(adapter(window.localStorage))

const enhancer = compose(
  persistState(storage, STORAGE_KEY)
)

// combine our store with our persistence strategy
export const store = createStore(reducer, enhancer)

```

---


### Component or Router

```js
import { Provider } from 'react-redux'

// Inject the store via props
ReactDOM.render(
  <Provider store={store}>
    <Router> ... </Router>
  </Provider>
)

// Connect the reducer to the props for the Root component (or elsewhere)
import { connect } from 'react-redux'
export default connect(({ user}) => ({ user }))(Root)

// Optionaly define some action creators
export function setUser (user, jwt) {
    return { type: SET_USER, user, jwt }
}

// Dispatch the action where necessary
store.dispatch(setUser(user, jwt))

```

---

- Create the store
- Create reducers and action and combine them as required
- Pass the store as props

---

## Vue

Disclaimer: This comparison is not exactly fair. You can actually use redux with vue and it behaves simlarly to the above example using`vue-redux`. Here I am describing `vuex`.

---

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
````

---

```js
// Actions
export function login (username, password) {
  return auth(username, password).then(function (jwt) {
    store.commit('addJwt', jwt)
  })
}
```

---

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

---

### Component

```js
// main.js
new Vue({
  el: '#app',
  router,
  store, // injects the store into the components
  template: '<App/>',
  components: { App }
})

// Componend.vue
this.$store.getters.isLoggedIn
```



- Use `vuex` and `vuex-localstorae`
- explicitly defined `getters`which can compute values
- mutations are handled through `store.commit`

---