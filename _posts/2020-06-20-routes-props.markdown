---
layout: post
title: Routes and Props in React
date: 2020-06-20 00:00:00 -0400
categories:
---

[React Router](https://reacttraining.com/react-router/) is a powerful library that allows React developers to intuitively and quickly handle navigation within their web applications. No longer confined to a single URL, an app using React Router can in fact be composed of different HTML pages that, in the source code, are written almost as if they were plain React components.

One important nuance of React Routes, however, is often glossed over in tutorials, and even the official documentation is not terribly clear about it: How do we pass props into our routes? It's a very basic question, and the unsuspecting newcomer to React Router (myself included) could reasonably expect something like the following to work:

```
/* WARNING: BROKEN CODE */

import React, { Component } from 'react'
import { BrowserRouter as Router, Route } from 'react-router-dom'
import Login from './components/Login.js'

class App extends Component {

  /* . . . */

  render() {
    return (
      <Router>
        <React.Fragment>
          <Route exact path="/login"
                 component={Login}
                 handleLogin={this.handleLogin}
                 handleSignup={this.handleSignup} />
        </React.Fragment>
      </Router>
    )
  }
}

export default App
```

Here we have an `App` component that has a child component of `Login`, which we are trying to assign to the `/login` route. `Login` takes two props, `handleLogin` and `handleSignup`, which are functions that will be called back up to `App` when a user tries to log in or sign up, respectively.

The problem here is that the `Login` component isn't actually receiving the `handleLogin` or `handleSignup` props. Instead, the `Route` component itself is receiving the props, which promptly disappear into the ether when the `/login` page is rendered.

If we are trying to pass props into our React Routes in the above way, we must make two changes:

1. Use the `render` prop instead of the `component` prop.
2. Wrap our components as if they were functional components.

Here would be the easiest way to make these changes:

```
<Route exact path="/login"
  render={ (props) => (<Login {...props}
    handleLogin={this.handleLogin}
    handleSignup={this.handleSignup}
  />) }
/>
```

What's happening here? Because `render` allows (expects, actually) a functional component, we can wrap our actual `Login` component inside of an arrow function that takes in some props as an argument. By passing those props into a spread operator, we can not only add in our custom `handleLogin` and `handleSignup` props, but we also have access to [the three default Route props](https://reacttraining.com/react-router/web/api/Route/route-props) (`match`, `location`, and `history`), which can aid further in our navigation.

Alternatively, we can write a separate function that handles the props for us. We can then call this function inside our `Route` component a bit more cleanly, without having to define our custom props inline:

```
import React, { Component } from 'react'
import { BrowserRouter as Router, Route } from 'react-router-dom'
import Login from './components/Login.js'

class App extends Component {

/* . . . */

const Login = (props) => {
  return (
    <Login
      {...props}
      handleLogin={this.handleLogin.bind(this)}
      handleSignup={this.handleSignup.bind(this)}
    />
  )
}

render() {
    return (
      <Router>
        <React.Fragment>
          <Route exact path="/login" render={Login} />
        </React.Fragment>
      </Router>
    )
  }
}

export default App
```

In addition to allowing us to pass our own props into our components, the `render` prop also has a performance edge over the `component` prop. With `component`, the component is instantiated each and every time that the route corresponding to that component is rendered. In other words, the mounting → unmounting → remounting life cycle is executed over and over while our user is navigating our React app. The `render` prop, by contrast, takes in a functional component, which is merely _evaluated_ on each render, _sans_ lifecycle methods.

Because the `render` prop is less expensive computationally and allows the passage of custom props into our components, I recommend using `render` whenever you are incorporating React Router into your project.

<h2 class="post-subheading">Further reading</h2>
- [How to pass props to React routes components](https://medium.com/alturasoluciones/how-to-pass-props-to-routes-components-29f5443eee94)
- [Pass props to a component rendered by React Router](https://tylermcginnis.com/react-router-pass-props-to-components/)
- [How to pass props to components when you use <Route components={{}} /> attribute ?](https://github.com/ReactTraining/react-router/issues/4105#issuecomment-287262726)
- [How to pass props to components in React](https://www.robinwieruch.de/react-pass-props-to-component/) (especially "Pass props with React Router")
- [react router difference between component and render](https://stackoverflow.com/questions/48150567/react-router-difference-between-component-and-render)
