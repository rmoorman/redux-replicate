# redux-replicate

[![build status](https://img.shields.io/travis/loggur/redux-replicate/master.svg?style=flat-square)](https://travis-ci.org/loggur/redux-replicate) [![npm version](https://img.shields.io/npm/v/redux-replicate.svg?style=flat-square)](https://www.npmjs.com/package/redux-replicate)
[![npm downloads](https://img.shields.io/npm/dm/redux-replicate.svg?style=flat-square)](https://www.npmjs.com/package/redux-replicate)

Store enhancer for [`redux`](https://github.com/rackt/redux) designed to easily replicate stores before and/or after `dispatch`.


## Installation

```
npm install redux-replicate --save
```


## Usage

```js
replicate (String storeKey, Object ...replicators)
```

This package exports a single function which returns a [`redux`](https://github.com/rackt/redux) store enhancer.  The first argument should be the name of your store, and the rest of the arguments should be replicators.  The enhancer adds the following 2 methods to the `store` object:

- `setState (Object state)` - Extends the current state with `state`, similarly to React's `setState` method.

- `setKey (String key)` - If the `key` doesn't match the current `storeKey`, sets the `storeKey` and then calls the `init` method for each replicator.  This is useful when you want to use different stores depending on values in other stores.


## Replicators

A replicator should be an object with at least one of the following keys as functions:

- `init (String storeKey, Object store)` - Called when initializing the store.  You can, for example, use the `store.setState` method here to asynchronously update the state of the store.

- `preDispatch (String storeKey, Object store, Object action)` - Called immediately before some `action` is dispatched.

- `postDispatch (String storeKey, Object store, Object action)` - Called immediately after some `action` is dispatched.


## Example replicator

See [`redux-replicate-localforage`](https://github.com/loggur/redux-replicate-localforage), a replicator that persists the state of your store(s) locally.


## Example using `compose`

```js
import { createStore, combineReducers, compose } from 'redux';
import replicate from 'redux-replicate';
import localforageReplicator from 'redux-replicate-localforage';
import reducers from './reducers';

const initialState = {
  wow: 'such storage',
  very: 'cool'
};

const storeKey = 'superCoolStorageUnit';
const replication = replicate(storeKey, localforageReplicator);
const create = compose(replication)(createStore);
const store = create(combineReducers(reducers), initialState);
```


## Example using [`react-redux-provide`](https://github.com/loggur/react-redux-provide)

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { assignProviders } from 'react-redux-provide';
import provideMap from 'react-redux-provide-map';
import logger from 'redux-logger';
import replicate from 'redux-replicate';
import localforageReplicator from 'redux-replicate-localforage';
import App from './components/App';

const coolMap = {
  ...provideMap('coolMap', 'coolItem'),
  middleware: logger,
  enhancer: replicate('coolMap', localforageReplicator)
};

assignProviders({ coolMap }, { App });

ReactDOM.render(<App/>, document.getElementById('root'));
```
