ReduxAsyncConnect for React Router
============
[![npm version](https://img.shields.io/npm/v/redux-connect.svg?style=flat-square)](https://www.npmjs.com/package/react-redux-async-connect)


How do you usually request data and store it to redux state?
You create actions that do async jobs to load data, create reducer to save this data to redux state,
then connect data to your component or container.

Usually it's very similar routine tasks.

Also, usually we want data to be preloaded. Especially if you're building universal app,
or you just want pages to be solid, don't jump when data was loaded.

This package consist of 2 parts: one part allows you to delay containers rendering until some async actions are happening.
Another stores your data to redux state and connect your loaded data to your container.

## Notice

This is a fork and refactor of [redux-async-connect](https://github.com/Rezonans/redux-async-connect)

## Installation & Usage

Using [npm](https://www.npmjs.com/):

`$ npm install --save react-redux-async-connect`

```js
import { Router, browserHistory } from 'react-router';
import { ReduxAsyncConnect, asyncConnect, reducer as reduxAsyncConnect } from 'react-redux-async-connect'
import React from 'react'
import { render } from 'react-dom'
import { createStore, combineReducers } from 'redux';

// 1. Connect your data, similar to react-redux @connect
@asyncConnect([{
  key: 'lunch',
  promise: ({ params, helpers }) => Promise.resolve({ id: 1, name: 'Borsch' })
}])
class App extends React.Component {
  render() {
    // 2. access data as props
    const lunch = this.props.lunch
    return (
      <div>{lunch.name}</div>
    )
  }
}

// 3. Connect redux async reducer
const store = createStore(combineReducers({ reduxAsyncConnect }), window.__data);

// 4. Render `Router` with ReduxAsyncConnect middleware
render((
  <Provider store={store} key="provider">
    <Router render={(props) => <ReduxAsyncConnect {...props}/>} history={browserHistory}>
      <Route path="/" component={App}/>
    </Router>
  </Provider>
), el)
```

### Server

```js
import { renderToString } from 'react-dom/server'
import { match, RoutingContext } from 'react-router'
import { ReduxAsyncConnect, loadOnServer, reducer as reduxAsyncConnect } from 'react-redux-async-connect'
import createHistory from 'history/lib/createMemoryHistory';
import { Provider } from 'react-redux';
import { createStore, combineReducers } from 'redux';
import serialize from 'serialize-javascript';

app.get('*', (req, res) => {
  const store = createStore(combineReducers({ reduxAsyncConnect }));

  match({ routes, location: req.url }, (err, redirect, renderProps) => {

    // 1. load data
    loadOnServer({ ...renderProps, store }).then(() => {

      // 2. use `ReduxAsyncConnect` instead of `RoutingContext` and pass it `renderProps`
      const appHTML = renderToString(
        <Provider store={store} key="provider">
          <ReduxAsyncConnect {...renderProps} />
        </Provider>
      )

      // 3. render the Redux initial data into the server markup
      const html = createPage(appHTML, store)
      res.send(html)
    })
  })
})

function createPage(html, store) {
  return `
    <!doctype html>
    <html>
      <body>
        <div id="app">${html}</div>

        <!-- its a Redux initial data -->
        <script dangerouslySetInnerHTML={{__html: `window.__data=${serialize(store.getState())};`}} charSet="UTF-8"/>
      </body>
    </html>
  `
}
```

## Usage with `applyRouterMiddleware`

Thanks to @mmahalwy for a good usage example
Pass custom `render` method to `ReduxAsyncConnect`, it can look like this:

```js
// on client
const component = (
  <Router
    render={(props) => (
      <ReduxAsyncConnect
        {...props}
        helpers={{ client }}
        filter={item => !item.deferred}
        render={applyRouterMiddleware(useScroll())}
      />
    )}
    history={history}
    routes={getRoutes(store)}
  />
);
```

Basically what you do is instead of using render method like:

```js
const render = props => <RouterContext {...props} />;
```

you use

```js
const render = applyRouterMiddleware(...middleware);
```
