# Routing & Auth

## Background

Some routes are private. when these are accessed we need to ensure the user is logged in, otherwise redirect them to the login page. In turn, once they have logged in successfully, we want to redirect them to somewhere private (normally the place they tried to reach initially).

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
- new example looks like this:

```jsx
<Route render={props => (
    authCheck ? (
      <Component {...props}/>
    ) : (
      <Redirect to={{
        pathname: '/login',
        state: { from: props.location }
      }}/>
    )
  )}/>
```



### Component

```jsx
<form onSubmit={onLogin}>
  <input name="email" type="text" value={email} onChange={setValue}></input>
  <input name="password" type="password" value={email} onChange={setValue}></input>
</form>
```

```js
setValue (evt) {
  const el = evt.target
  const name = el.getAttribute('name')

  this.setState({
    [name]: el.value
  })
}

onLogin (evt) {
  evt.preventDefault()
  
  login( this.state.email, this.state.password ).then(user => {
     hashHistory.push('/')
  })
}
```

- React deprecated the `valueLink` property
- We provide the setValue helper

## Vue

### Router

```js
const ensureLoggedIn = (to, from, next) => {
  if (!store.getters.isLoggedIn) {
    return next('/login')
  } else {
    return next() // go to the next handler
  }
}

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

- Based on plain JS

- Limited to 1 beforeEnter hook so you have to chain these manually

  ​

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



