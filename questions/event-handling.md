# How does event handling work? 

```jsx
import React, { Component } from 'react';

class Example extends Component {
  render() {
    return (
        <button onClick={activateLasers}>
            Activate Lasers
        </button>
    );
  }
}

export default Example;
```


Basically, handling events with React is in someway similar to handling events on DOM elements. The documentation tells us   these main differences:

- React events are named using camelCase, rather than lowercase.
- With JSX you pass a function as the event handler, rather than a string.

But I'll try to answer this questions in a more low level way. By looking the source code of React and see its functionability. For this task, I installed the source code of *React* in my computer and looking for this answer.

React handles events with something called event delegation. When we render something like the code above, React installs a DOM event listener where the component is rendering. This will happen every time a component is mounted or updated, according to
React lifecycles.

For handling these events React uses something called *Synthetic Events*. Syntentic events are defined by an interface, you can see its implementation here:

```javascript
    onClick?: MouseEventHandler<T>;
    onLoad?: ReactEventHandler<T>;
    onSubmit?: FormEventHandler<T>;
```

```typescript

    type MouseEventHandler<T = Element> = EventHandler<MouseEvent<T>>;

    type EventHandler<E extends SyntheticEvent<any>> = { bivarianceHack(event: E): void }["bivarianceHack"];

    interface SyntheticEvent<T = Element, E = Event> extends BaseSyntheticEvent<E, EventTarget & T, EventTarget> {}

```
Event handlers will be passed instances of SyntheticEvent, a cross-browser wrapper around the browser’s native event. In case of
the code above we see a MouseEvent being passed.

It has the same interface as the browser’s native event, including stopPropagation() and preventDefault(), except the events work identically across all browsers.


## Handling a native event
React renders the application to the DOM, setting some event listeners in the document root. And then something has to happen (for example, clicking a bottom, submitting a form) and a native event is triggered. This will fire the top-level DOM event listener responsible for handling that event.

```javascript
function registerReactDOMEvent(
  target: EventTarget | ReactScopeInstance,
  domEventName: DOMEventName,
  isCapturePhaseListener: boolean,
): void {
  if ((target: any).nodeType === ELEMENT_NODE) {
    // Do nothing. We already attached all root listeners.
  } else if (enableScopeAPI && isReactScope(target)) {
    // Do nothing. We already attached all root listeners.
  } else if (isValidEventTarget(target)) {
    const eventTarget = ((target: any): EventTarget);
    // These are valid event targets, but they are also
    // non-managed React nodes.
    listenToNativeEventForNonManagedEventTarget(
      domEventName,
      isCapturePhaseListener,
      eventTarget,
    );
  } else {
    invariant(
      false,
      'ReactDOM.createEventHandle: setter called on an invalid ' +
        'target. Provide a valid EventTarget or an element managed by React.',
    );
  }
}
```

## API level 3
React defines Events as a common interface which use the DOM Level 3 Events API.

```typescript
/**
 * @interface Event
 * @see http://www.w3.org/TR/DOM-Level-3-Events/
 */
const EventInterface = {
  eventPhase: 0,
  bubbles: 0,
  cancelable: 0,
  timeStamp: function(event) {
    return event.timeStamp || Date.now();
  },
  defaultPrevented: 0,
  isTrusted: 0,
};
export const SyntheticEvent = createSyntheticEvent(EventInterface);
```

## The `EventPluginHub`

Its main functionality is:

- Provides an interface for event plugins to be used.
- Runs through the injected plugins every time a new native event is received, collecting the SyntheticEvents returned before dispatching them all.


```typescript
/**
 * Dispatches an event and releases it back into the pool, unless persistent.
 *
 * @param {?object} event Synthetic event to be dispatched.
 * @private
 */
const executeDispatchesAndRelease = function(event: ReactSyntheticEvent) {
  if (event) {
    executeDispatchesInOrder(event);

    if (!event.isPersistent()) {
      event.constructor.release(event);
    }
  }
};
const executeDispatchesAndReleaseTopLevel = function(e) {
  return executeDispatchesAndRelease(e);
};
```

Basically it gathers all SyntheticEvents and their dispatch configuration and store them in the queue. Execute all dispatches for all events in the queue, effectively clearing or releasing it.

And that's is how *event handling* works in React. 