# Why we need to pass a function to setState()?


The function `setState()` basically, defines a subset of the state. In the documentation of `setState()` it says that there is no guarantee that by using `this.state` will be immediately updated, so
accessing `this.state` after calling this method may return the old value.
 
There is no guarantee that calls to `setState` will run synchronously, as they may eventually be batched together.  You can provide an optional callback that will be executed when the call to setState is actually completed.

When a function is provided to `setState`, it will be called at some point in the future (not synchronously). It will be called with the arguments (state, props, context). These values can be different from this because the function may be called after receiveProps but before `shouldComponentUpdate`, and this new state, props, and context will not yet be assigned to this.
 
The React component inherits from React.Component, and setState is the React.Component method, so setState belongs to its prototype method for the component.
```typescript
Component.prototype.setState = function(partialState, callback) {
  invariant(
    typeof partialState === 'object' ||
      typeof partialState === 'function' ||
      partialState == null,
    'setState(...): takes an object of state variables to update or a ' +
      'function which returns an object of state variables.',
  );
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};

```

`partialState` as the name suggests that it does not affect the meaning of the original state.

When the setState is called, the `enqueueSetState` method is actually called. 

Basically, here `updater` is an object containing methods to update the DOM, calling `setState` especifically.

# But what does `enqueueSetState` do?

It sets a subset of the state. Where we pass:
1. The `publicInstance`, the instance that should rerender when the state updates.
2. The `partialState`. This being the next partial state to be merged with state.
3. The function `callback` called after component is updated. *The answer to the question*
4. And the name of the calling function in the public API, in this case `setState`.

```javascript
enqueueSetState: function(
    publicInstance,
    partialState,
    callback,
    callerName,
  ) {
    warnNoop(publicInstance, 'setState');
  }
```
TODO
# What are recieveProps and shouldComponentUpdate?

# Why does enqueueSetState receive many arguments but only uses one on its body? What happens with the callback?

# What does warnNoop do?
