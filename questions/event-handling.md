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