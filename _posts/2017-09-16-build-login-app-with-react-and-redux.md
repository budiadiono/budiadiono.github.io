---
layout: single
title:  "Build simple login application with React Native and Redux"
date:   2017-09-15 20:31:00 +0700
permalink: '/post/react/build-login-app-with-react-and-redux'
categories: 
  - react
---

{% include toc title="Contents" %}

This is the part two of [**Meet React Native**](/post/react/meet-react-native).

To create a simple login application, I need a state to hold the login status and also the user identity.
I use [**Redux**](http://redux.js.org/) to handle the state management.

First, install the packages:

```bash
yarn add redux react-redux redux-actions redux-thunk --save
```

Well, this is my first experience working with **Redux**. As I know there are two things to make mutation in **Redux**:

1. **Actions** -- Dispacthing plain object that contains `type` and `payload`.
2. **Reducer** -- Consuming **Actions** and return new application state. This state will be reactive for entire application.

I found nice example of how to use React Native and Redux [here](https://github.com/alinz/example-react-native-redux/tree/master/Counters).
This simple login application was adapted from that code with some improvements.

## Structure

To make this development fun, I build the structure like this:

```bash
root
└── App
   ├── components       # common components like button, etc..
   |  ├── index.js      #  exporting all components
   |  ├── Button.js     #    Button component
   |  └── ...           #    other components...
   ├── screens          # screens or views
   |  ├── index.js      #  exporting all screens
   |  ├── main.js       #    main screen
   |  ├── login.js      #    login screen
   |  └── ...           #    other screens..
   ├── store            # store and modules
   |  ├── modules       # each module contains it's own reducers and actions
   |  |   ├── index.js  #   exporting all modules actions and reducers 
   |  |   ├── app       #     app module
   |  |   ├── user      #     user module
   |  |   └── ...       #     other modules...
   |  └── index.js      # exporting store that built from modules
   └── index.js         # root application

```

## App/store/modules

For complex application, we might want to split the state into several modules. To demonstrate this case, I have two modules in this application:

1. **user** is to handle user identity and the login activity.
2. **app** is to handle loading state that can be use for many purposes.

Each module have 4 files:

### constants.js

It's simply contains only the name of action types.

*App/store/modules/user/constants.js*

```javascript
export const LOGIN = 'USER/LOGIN'
export const LOGOUT = 'USER/LOGOUT'
```

All types should be prefixed with the module name (`USER/`) it's to prevent collision with other module.

So *App/store/modules/app/constants.js* will be prefixed with `APP/`:

```javascript
export const SET_LOADING = 'APP/SET_LOADING'
```
   
### actions.js

Contains set of actions functions. We make our app logic here.
Each action can interact with another module actions

Each action should returns:

```
{
  type: <name of action defined in constants.js>,
  payload: <the actual payload -- it's optional>
}
```

Or, for async call, action should returns:

```
(dispatch, getState) => {
  // * dispatch is a callback to dispacth the action type and payload
  //   example: 
  //     dispatch({type: LOGIN, payload: {userid: 'superman', name: 'Clark Kent'}})
  //
  // * getState is a function to get the current state. 
  //   To get the current state you have to invoke getState()
}
```

Async call is supported by **thunk** middleware from [redux-thunk](https://github.com/gaearon/redux-thunk), 
that if you return a function from action instead of an object, thunk middleware
will kicks in and calls your function with `dispatch` and `getState` argument.

*App/store/modules/app/actions.js*

```javascript
import * as types from './constants'

/**
* Set loading status on/off
* @param {boolean} yes Loading status
*/
export const loading = (yes: boolean = true) => {
  return {
    type: types.SET_LOADING,
    payload: yes
  }
}
```

*App/store/modules/user/actions.js*

```javascript
import * as types from './constants'
import { actions } from '../'

/**
* Sign in.
* @param {string} username 
* @param {string} password
*/
export const login = (username: string, password: string) => {
  // async call
  return dispatch => {
    // turn loading animation on
    // by dispacthing `loading` action from module `app`.
    // yes, each action can interact with another module actions.
    dispatch(actions.app.loading())
    
    // simulate ajax login
    // in real world you can use `fetch` to make ajax request.
    setTimeout(() => {
      if (username === 'admin' && password === 'secret') {
        dispatch({
          type: types.LOGIN,
          payload: {
            userId: username,
            fullName: 'Clark Kent'
          }
        })
      }
    
      // turn loading animation off
      dispatch(actions.app.loading(false))
    }, 3000)
  }
}

/**
* Sign out.
*/
export const logout = () => {
  // direct/sync call
  return {
    type: types.LOGOUT
  }
}

```


### reducer.js
Is to handle how state being mutate depending of what actions disptached as described in the [official doc](http://redux.js.org/docs/basics/Reducers.html).

By default, each reducer should returns `handleActions` from `redux-actions`.
A `handleActions` is a helper function. Instead of using a switch case statement,
we just use the regular map with function state attach to it keyed by the name of action type.

In reducer, the initial state and also type of state are defined.
We should also export the type of state to support the *type safe* while coding.
   
*App/store/modules/app/reducer.js*

```javascript
import { handleActions } from 'redux-actions'
import { SET_LOADING } from './constants'

// exporting type of state for type safe
export type AppState = {
  loading: boolean
}

const initialState: AppState = {
  loading: false
}

// handle actions
export default handleActions(
  {
    [SET_LOADING]: (state: AppState = initialState, action): AppState => {
      return {
        loading: action.payload
      }
    }
  },
  initialState
)
```

*App/store/modules/user/reducer.js*

```javascript
import { handleActions } from 'redux-actions'
import { LOGIN, LOGOUT } from './constants'

export type UserState = {
  loggedIn: boolean,
  userId: string,
  fullName: string
}

const initialState: UserState = {
  loggedIn: false,
  userId: '',
  fullName: ''
}

export default handleActions(
  {
    [LOGIN]: (state: UserState = initialState, action): UserState => {
      const p = action.payload
      return {
        loggedIn: true,
        userId: p.userId,
        fullName: p.fullName
      }
    },

    [LOGOUT]: (): UserState => {
      return {
        loggedIn: false
      }
    }
  },
  initialState
)
```


### index.js

Combine and re-export of what each module have. It should always have same pattern like so:

*App/store/modules/app/index.js*

```javascript
import reducer from './reducer'
import * as actions from './actions'

export const app = { reducer, actions }
export { AppState } from './reducer'
```

*App/store/modules/user/index.js*

```javascript
import reducer from './reducer'
import * as actions from './actions'

export const user = { reducer, actions }
export { UserState } from './reducer'
```


## App/store/module/index.js

Finaly, all modules combined here.

```javascript
import { UserState, user } from './user'
import { AppState, app } from './app'

/**
 * Root states.
 */
export type States = {
  app: AppState,
  user: UserState
}

/**
 * Root reducers.
 */
export const reducers = {
  app: app.reducer,
  user: user.reducer
}

/**
 * Root actions.
 */
export const actions = {
  app: app.actions,
  user: user.actions
}

export { app, user }
```

Now modules is ready to process by store.

## App/store/index.js

Re-export modules actions and state types, thunkify, and build a `createStore` function.

```javascript
import {
  createStore as _createStore,
  applyMiddleware,
  combineReducers
} from 'redux'
import thunk from 'redux-thunk'
import { reducers, actions } from './modules'

/**
 * Root states types.
 */
export { States } from './modules'

// Apply thunk middleware
const middleware = applyMiddleware(thunk)

/**
 * Create app store.
 */
const createStore = (data: Object = {}) => {
  return _createStore(combineReducers(reducers), data, middleware)
}

export { createStore, actions }
```

## App/components

This folder will contains all plain UI components needed by application. 
For example I created my custom button in `Button.js`.

*App/components/Button.js -- copied from https://github.com/alinz/example-react-native-redux/blob/master/Counters/src/components/Button.js*

```javascript
import React from 'react'
import { StyleSheet, Text, TouchableOpacity, View } from 'react-native'

const styles = StyleSheet.create({
  button: {
    height: 50,
    padding: 20,
    backgroundColor: 'lightgray',
    alignItems: 'center',
    justifyContent: 'center',

    margin: 5
  }
})

type ButtonProps = {
  children?: any,
  onPress: () => void
}

export const Button = (props: ButtonProps) => {
  const { children, onPress } = props

  return (
    <TouchableOpacity onPress={onPress} style={styles.button}>
      <View>
        <Text>{children}</Text>
      </View>
    </TouchableOpacity>
  )
}
```

As usualy, everything exported by `index.js`:

*App/components/index.js*

```javascript
export { Button } from './Button'
```


## App/screens

This folder contains screens. Screen is a React component that connected to store.
We use `connect` function from `react-redux` to inject states and actions from modules
into component's `props`.

*App/screens/login.js*

```javascript
import React, { Component } from 'react'
import {
  StyleSheet,
  View,
  Text,
  TextInput,
  ActivityIndicator
} from 'react-native'
import { connect } from 'react-redux'
import { actions, States } from '../store'
import { Button } from '../components'

/**
 * A login component that display username and password text field.
 * Loading indicator will show up when login is in process.
 * 
 * @class App
 * @extends {Component}
 */
class App extends Component {
  constructor(props) {
    super(props)

    // init local state
    this.state = {
      username: 'admin',
      password: 'secret'
    }
  }

  render() {
    const { loading, doLogin } = this.props

    // show only loading indicator if loading state is true
    if (loading) {
      return <ActivityIndicator />
    }


    // display login screen
    return (
      <View style={styles.container}>
        <Text>Login</Text>
        <TextInput
          onChangeText={username => this.setState({ username })}
          value={this.state.username}
        />
        <TextInput
          onChangeText={password => this.setState({ password })}
          value={this.state.password}
        />
        <Button
          onPress={() => {
            doLogin(this.state.username, this.state.password)
          }}
        >
          Login
        </Button>
      </View>
    )
  }
}

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    justifyContent: 'center'
  }
})

/**
 * Login screen.
 */
export const Login = connect(
  
  // inject states
  (state: States) => ({
    
    // props.loading -> modules.app.loading
    loading: state.app.loading
  }),
  
  // inject actions
  dispatch => ({

    // props.doLogin -> modules.login.login()
    doLogin: (username, password) =>
      dispatch(actions.user.login(username, password))
  })
)(App)
```

*App/screens/main.js*

```javascript
import React, { Component } from 'react'
import { View, Text } from 'react-native'
import { connect } from 'react-redux'
import { actions, States } from '../store'
import { Login } from './login'
import { Button } from '../components'

/**
 * Main component. Display greeting when user is logged in,
 * otherwise it will display the login screen.
 * 
 * @class App
 * @extends {Component}
 */
class App extends Component {
  render() {
    const { doLogout, loggedIn, fullName } = this.props

    // Display login screen when user is not logged in
    if (!loggedIn) {
      return (
        <View>
          <Text>Awesome Project</Text>
          <Login />
        </View>
      )
    }

    // Display greeting with user full name displayed
    return (
      <View>
        <Text>Welcome {fullName}!</Text>
        <Button
          onPress={() => {
            doLogout()
          }}
        >
          Logout
        </Button>
      </View>
    )
  }
}

export const Main = connect(

  // inject states to props
  (state: States) => ({
    loggedIn: state.user.loggedIn,
    fullName: state.user.fullName
  }),
  
  // inject actions to props
  dispatch => ({
    doLogout: () => dispatch(actions.user.logout())
  })
)(App)
```

As usualy, everything exported by `index.js`. 
But this time I only import the `main` screen since we only need the main screen to display
to the root application. The `login` screen already tackle by `main` screen it self.

*App/screens/index.js*

```javascript
export { Main } from './main'
```

## App/index.js
This is the final step.

The root application now is in `App/index.js`. You can delete `App.js` and create `App/index.js` then use this code:

*App/index.js*

```javascript
import React from 'react'
import { Provider } from 'react-redux'

import { Main } from './screens'
import { createStore } from './store'

const store = createStore()

/**
 * Root application.
 */
const App = () => {
  return (
    <Provider store={store}>
      <Main />
    </Provider>
  )
}

export default App
```

Done! It's time to test it out.

```bash
yarn start
```

Turn on your emulator then press `a`. And.. voilà! 

![Running in emulator](/assets/images/2017-09-15-meet-react-native/app-running.gif)

What a nice experience working with **React Native** and **Redux**. 
The full source code in [this](https://github.com/budiadiono/meet-react-native-redux) repository.


Any suggestions are welcome. Thanks for reading!


