# Why we need to pass a function to setState()?


The function `setState()` basically, defines a subset of the state. In the documentation of `setState()` it says that there is no guarantee that by using `this.state` will be immediately updated, so
accessing `this.state` after calling this method may return the old value.
 
There is no guarantee that calls to `setState` will run synchronously,
as they may eventually be batched together.  You can provide an optional
callback that will be executed when the call to setState is actually
completed.

When a function is provided to `setState`, it will be called at some point in
the future (not synchronously). It will be called with the up to date
component arguments (state, props, context). These values can be different
from this because the function may be called after receiveProps but before `shouldComponentUpdate`, and this new state, props, and context will not yet be
assigned to this.
 
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


The reason behind for this is that `setState()` is an asynchronous operation. React batches state changes for performance reasons, so the state may not change immediately after `setState()` is called. That means you should not rely on the current state when calling `setState()` since you can't be sure what that state will be. The solution is to pass a function to `setState()`, with the previous state as an argument. By doing this you can avoid issues with the user getting the old state value on access due to the asynchronous nature of `setState()`.