# Handling raw DOM elements

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

## Vue 

```
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