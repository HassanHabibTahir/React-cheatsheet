# Cheat Sheet

## JSX

JSX stands for JavaScript XML. It is simply a syntax extension of JavaScript. It allows us to directly write HTML in React (within JavaScript code). It is easy to create a template using JSX in React, but it is not a simple template language instead it comes with the full power of JavaScript

```javascript
import logo from "./logo.svg";
import "./App.css";
function App() {
  const element = <h1>Hello, world!</h1>;
  return <div>{element}</div>;
}
export default App;
```

## Props

In ReactJS, the props are a type of object where the value of attributes of a tag is stored. The word “props” implies “properties”, and its working functionality is quite similar to HTML attributes.
Basically, these props components are read-only components.

```javascript
function Introduction(props) {
  return <h1>Hello, I’m {props.name}!</h1>;
}
function Introduction({ name }) {
  return <h1>Hello, I’m {name}!</h1>;
}
const element = <Introduction name="The Doctor" />;
```

## State

What is state?
React has another special built-in object called state, which allows components to create and manage their own data. So unlike props, components cannot pass data with state, but they can create and manage it internally.

From Component State to Hooks

# From Component State to Hooks

We're going to start with a [super simple counter](https://github.com/stevekinney/simple-counter).

Out of the box, it doesn't have a lot going on.

```js
class Counter extends Component {
  render() {
    return (
      <main className="Counter">
        <p className="count">0</p>
        <section className="controls">
          <button>Increment</button>
          <button>Decrement</button>
          <button>Reset</button>
        </section>
      </main>
    );
  }
}
```

Let's get it wired up as a fun warmup exercise.

## Getting the Basic Component Wired Up

We'll start with a constructor method that sets the component state.

```js
constructor(props) {
  super(props);
  this.state = {
    count: 0,
  };
}
```

We'll use that state in the component.

```js
render() {
  const { count } = this.state;

  return (
    <main className="Counter">
      <p className="count">{count}</p>
      <section className="controls">
        <button>Increment</button>
        <button>Decrement</button>
        <button>Reset</button>
      </section>
    </main>
  );
}
```

Alright, now we'll implement the methods to incrementm, decrement, and reset the count.

```js
increment() {
  this.setState({ count: this.state.count + 1 });
}

decrement() {
  this.setState({ count: this.state.count - 1 });
}

reset() {
  this.setState({ count: 0 });
}
```

We'll add those methods to the buttons.

```js
<button onClick={this.increment}>Increment</button>
<button onClick={this.decrement}>Decrement</button>
<button onClick={this.reset}>Reset</button>
```

We need to bind those event listeners because everything is terrible.

```js
constructor(props) {
  super(props);
  this.state = {
    count: 3,
  };

  this.increment = this.increment.bind(this);
  this.decrement = this.decrement.bind(this);
  this.reset = this.reset.bind(this);
}
```

## Component State Pop Quiz

### Asyncronous Updates and Queuing

Okay, let's say we refactored `increment()` as follows:

```js
increment() {
  this.setState({ count: this.state.count + 1 });
  this.setState({ count: this.state.count + 1 });
  this.setState({ count: this.state.count + 1 });

  console.log(this.state.count);
}
```

Two questions:

1. What will be logged to the console?
2. What will the new value be?

### Using a Function as an Argument

`this.setState` also takes a function. This means we could refactor `increment()` as follows.

```js
this.setState((state) => {
  return { count: state.count + 1 };
});
```

If we wanted to show off, we could use destructuring to make it evening cleaner.

```js
increment() {
  this.setState(({ count }) => {
    return { count: count + 1 };
  });
}
```

There are some potentally cool things we could do here. For example, we could add some logic to our component.

**(Live Coding Starts Here)**

```js
this.setState((state, props) => {
  if (state.count >= 10) return;
  return { count: state.count + 1 };
});
```

Let's stay, we wanted to add in a maximum count as a prop.

```js
render(<Counter max={10} />, document.getElementById("root"));
```

```js
this.setState((state) => {
  if (state.count >= this.props.max) return;
  return { count: state.count + 1 };
});
```

It turns out that we can actually have a second argument in there as well—the `props`.

```js
this.setState((state, props) => {
  if (state.count >= props.max) return;
  return { count: state.count + 1 };
});
```

Oh wait—what if we did this three times?

```js
increment() {
  this.setState((state, props) => {
    if (state.count >= props.max) return;
    return { count: state.count + 1 };
  });
  this.setState((state, props) => {
    if (state.count >= props.max) return;
    return { count: state.count + 1 };
  });
  this.setState((state, props) => {
    if (state.count >= props.max) return;
    return { count: state.count + 1 };
  });
}
```

Oh, that's interesting.

The other thing we can do is pull that function out of the component. This makes it _way_ easier to unit test.

```js
const increment = (state, props) => {
  if (state.count >= props.max) return;
  return { count: state.count + 1 };
};
```

```js
increment() {
  this.setState(increment);
}
```

### Callbacks

`this.setState` takes a second argument in addition to either the object or function. This function is called after the state change has happened.

Here is the simplest possible implementation.

```js
this.setState(increment, () => console.log("Callback"));
```

We can also do something like.

```js
this.setState(increment, () => console.log(this.state));
```

Here is a simple thing we might choose to do.

```js
this.setState(increment, () => (document.title = `Count: ${this.state.count}`));
```

### Using LocalStorage in a Side Effect

Let's make a little helper function.

```js
const getStateFromLocalStorage = () => {
  const storage = localStorage.getItem("counterState");
  if (storage) return JSON.parse(storage);
  return { count: 0 };
};
```

We can then use a callback to set `localStorage` when the state changes.

```js
this.setState(increment, () =>
  localStorage.setItem("counterState", JSON.stringify(this.state))
);
```

Let's pull that out along with increment.

```js
const storeStateInLocalStorage = () => {
  localStorage.setItem("counterState", JSON.stringify(this.state));
};
```

```js
increment() {
  this.setState(increment, storeStateInLocalStorage);
}
```

It doesn't work. It's a bummer. It would be great if the callback function got a copy of the state, but it doesn't. We could wrap it into a function and then pass the state in.

We could handle this a few ways.

We could use an anonymous function and then pass it in as an argument.

```js
const storeStateInLocalStorage = (state) => {
  localStorage.setItem("counterState", JSON.stringify(state));
};
```

```js
increment() {
  this.setState(increment, () => storeStateInLocalStorage(this.state));
}
```

Alternatively, if we're willing to give up on arrow functions, we can use `bind`.

```js
function storeStateInLocalStorage() {
  localStorage.setItem("counterState", JSON.stringify(this.state));
}
```

```js
increment() {
  this.setState(increment, storeStateInLocalStorage.bind(this));
}
```

Lastly, we can just put it onto the class component itself.

```js
storeStateInLocalStorage() {
  localStorage.setItem('counterState', JSON.stringify(this.state));
}

increment() {
  this.setState(increment, this.storeStateInLocalStorage);
}
```

This is probably your best bet.

## Refactoring to Hooks

Hooks are a new pattern that allow us to write a lot less code. Get ready to delete some code.

Let's start by deleting everything but the render method.

```js
const Counter = ({ max }) => {
  const [count, setCount] = React.useState(0);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(0);

  return (
    <main className="Counter">
      <p className="count">{count}</p>
      <section className="controls">
        <button onClick={increment}>Increment</button>
        <button onClick={decrement}>Decrement</button>
        <button onClick={reset}>Reset</button>
      </section>
    </main>
  );
};
```

So, what _don't_ we have to do here?

- We don't have to `bind` anything.
- We dont need a reference to this.
- We don't need a `constructor` at all.

### Running Some of Our Previous Experiments

What if we tripled up again?

```js
const increment = () => {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
};
```

Okay, so that makes sense.

It also turns out that `useState` setters can take functions too.

```js
const increment = () => {
  setCount((c) => c + 1);
};
```

Unlike using values, using functions also works the same way as it does with `this.setState`.

```js
const increment = () => {
  setCount((c) => c + 1);
  setCount((c) => c + 1);
  setCount((c) => c + 1);
};
```

There is an important difference. You get _only_ the state in this case. There is no second argument with the props. That said, we have them in scope.

They also do _not_ support callback functions like `this.setState`. Later on, we'll use `useEffect` to trigger side effects based on state changes.

Earlier with `this.setState`, we ended up returning `undefined` if our count had hit the max. What if we did something similar here?

```js
setCount((c) => {
  if (c >= max) return;
  return c + 1;
});
```

Oh—it explodes after ten. This is core to the difference between how `useState` and `this.setState` works.

With `this.setState`, we're giving the component that object of values that it needs to update. With `useState`, we've got a dedicated function to change a particular piece of state.

How can we fix this?

```js
setCount((c) => {
  if (c >= max) return c;
  return c + 1;
});
```

## A Brief Introduction to useEffect

We're going to go a bit deeper into `useEffect`, but let's do the high level now.

Use effect allows us to implement some kind of side effect in our component outside of the changes to state and props triggering a new render.

This is useful for a ton of reasons:

- Storing stuff in `localStorage`.
- Making AJAX requests.

### Implementing `localStorage`

Let's get the basic set up in place here.

Here is a reminder of that function of getting from `localStorage`.

```js
const getStateFromLocalStorage = () => {
  const storage = localStorage.getItem("counterState");
  if (storage) return JSON.parse(storage);
  return { count: 0 };
};
```

We'll read the count property from `localStorage`.

```js
const [count, setCount] = React.useState(getStateFromLocalStorage().count);
```

Now, we'll register a side effect.

```js
React.useEffect(() => {
  localStorage.setItem("counterState", JSON.stringify({ count }));
}, [count]);
```

## Events

Event handlers determine what action is to be taken whenever an event is fired. This could be a button click or a change in a text input.

Essentially, event handlers are what make it possible for users to interact with your React app. Handling events with React elements is similar to handling events on DOM elements, with a few minor exceptions.

```javascript
function ActionLink() {
  const handleClick = (e) => {
    e.preventDefault();
    console.log("The link was clicked.");
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

## What are synthetic events in React?

React implements a synthetic events system that brings consistency and high performance to React apps and interfaces. It achieves consistency by normalizing events so that they have the same properties across different browsers and platforms.

A synthetic event is a cross-browser wrapper around the browser’s native event. It has the same interface as the browser’s native event, including `stopPropagation()` and `preventDefault()`, except the events work identically across all browsers.

It achieves high performance by automatically using event delegation. In actuality, React doesn’t attach event handlers to the nodes themselves. Instead, a single event listener is attached to the root of the document. When an event is fired, React maps it to the appropriate component element.

## Call an inline function in an onClick event handler

Inline functions allow you to write code for event handling directly in JSX. See the example below:

```javascript
import React from "react";

const App = () => {
  return <button onClick={() => alert("Hello!")}>Say Hello</button>;
};

export default App;
```

This is commonly used to avoid the extra function declaration outside the JSX, although it can be less readable and harder to maintain if the content of the inline function is too much.

## Update the state inside an onClick event handler

Let’s say your React application requires you to update the local state in an onClick event handler. Here’s how to do that:

```javascript
import React, { useState } from "react";

const App = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
    </div>
  );
};

export default App;
```

In the example above, the value of `useState` is modified by the `Increment` and `Decrement` buttons, which have the `setCount`, an updater function inside the `onClick` event handler.

## Call multiple functions in an onClick event handler

The onClick event handler also allows you to call multiple functions.

```javascript
import React, { useState } from "react";

const App = () => {
  const [count, setCount] = useState(0);
  const sayHello = () => {
    alert("Hello!");
  };

  return (
    <div>
      <p>{count}</p>
      <button
        onClick={() => {
          sayHello();
          setCount(count + 1);
        }}
      >
        Say Hello and Increment
      </button>
    </div>
  );
};

export default App;
```

## Custom components and events in React

When it comes to events in React, only DOM elements are allowed to have event handlers. Take the example of a component called CustomButton with an onClick event. This button wouldn’t respond to clicks because of the reason above.

```javascript
import React from "react";

const CustomButton = ({ onPress }) => {
  return (
    <button type="button" onClick={onPress}>
      Click on me
    </button>
  );
};

const App = () => {
  const handleEvent = () => {
    alert("I was clicked");
  };
  return <CustomButton onPress={handleEvent} />;
};
export default App;
```

## Life Cycle

### Mounting

These methods are called in the following order when an instance of a component is being created and inserted into the DOM:

constructor()
static getDerivedStateFromProps()
render()
componentDidMount()

### Updating

An update can be caused by changes to props or state. These methods are called in the following order when a component is being re-rendered:

static getDerivedStateFromProps()
shouldComponentUpdate()
render()
getSnapshotBeforeUpdate()
componentDidUpdate()

### Unmounting

This method is called when a component is being removed from the DOM:

componentWillUnmount()

### Hooks;

### useState

### useEffect

This means useEffect runs on every render and thus the event handlers will unnecessarily get detached and reattached on each render.

```js
window.React.useEffect(() => {
  console.log(`adding listener ${count}`);
  window.addEventListener("click", listener);
});
```

### component will unmount

his is the optional cleanup mechanism for effects. Every effect may return a function that cleans up after it. This lets us keep the logic for adding and removing subscriptions close to each other. They’re part of the same effect!
the side-effect runs after every rendering.

```js
window.React.useEffect(() => {
  return () => {
    console.log(`removing listener ${count}`);
    window.removeEventListener("click", listener);
  };
});
```

```js
import { useEffect } from "react";
function RepeatMessage({ message }) {
  useEffect(() => {
    const id = setInterval(() => {
      console.log(message);
    }, 2000);
    return () => {
      clearInterval(id);
    };
  }, [message]);
  return <div>I'm logging to console "{message}"</div>;
}
```

### useEffect componentDidMount

the side-effect runs once after the initial rendering.

```js
import { useEffect } from "react";
function Greet() {
  let name = "Hassan Habib Tahir";
  const message = `Hello, ${name}!`; // Calculates output
  useEffect(() => {
    // Good!
    document.title = `Greetings to ${name}`; // Side-effect!
  }, []);
  return <div>{message}</div>; // Calculates output
}
```

callback is the function containing the side-effect logic. callback is executed right after changes were being pushed to DOM.
dependencies is an optional array of dependencies. useEffect() executes callback only if the dependencies have changed between renderings.

```js
useEffect(callback[, dependencies]);
```

### Component did update

```js
import { useEffect } from "react";
function MyComponent({ prop }) {
  const [state, setState] = useState();
  useEffect(() => {
    // Side-effect uses `prop` and `state`
  }, [prop, state]);
  return <div>....</div>;
}
```

### useMemo

useMemo is a React Hook that lets you cache the result of a calculation between re-renders.
The basic purpose of the useMemo hook is related to the fact that we try to avoid the unnecessary re-rendering of components and props in our program.
<b>Example 1</b>
Let’s make a simple application to demonstrate the use of the useMemo hook.

The program below has the following components:

<b>Increment</b> button: starts from 0 and increases the count by 1.

<b>Even num</b> button: starts from 2 and returns even numbers going forward.

An <b>evenNumDoouble()</b> function that returns the twice value of the even number.

```js
import React, { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);
  const [evenNum, setEvenNum] = useState(2);

  function evenNumDouble() {
    console.log("double");
    return evenNum * 2;
  }

  return (
    <div>
      <h2>Counter: {count}</h2>
      <h2>Even Numbers: {evenNum}</h2>
      <h2>even Number Double Value: {evenNumDouble()}</h2>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setEvenNum(evenNum + 2)}>Even Numbers</button>
    </div>
  );
}

export default Counter;
```

Explanation
When we click the button Even Numbers, the evenNumDouble() function is called because the state of evenNum is changed.

Clicking the Increment button also renders the evenNumDouble() function, although the count state does not change.

This means that every time the evenNumDouble() function is rendered unnecessarily (on the page), the code becomes less efficient. We will fix this with the useMemo hook below.

```js
import React, { useState, useMemo } from "react";

function Counter() {
  const [count, setCount] = useState(0);
  const [evenNum, setEvenNum] = useState(2);

  const memoHook = useMemo(
    function evenNumDouble() {
      console.log("double");
      return evenNum * 2;
    },
    [evenNum]
  );

  return (
    <div>
      <h2>Counter: {count}</h2>
      <h2>Even Numbers: {evenNum}</h2>
      <h2>even Number Double Value: {memoHook}</h2>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setEvenNum(evenNum + 2)}>Even Numbers</button>
    </div>
  );
}

export default Counter;
```

Explanation
In the code above, we set the output of the evenNumDouble() function into a constant memoHook.

This filters the function through the useMemo hook to only check if the specified variable (evenNum in this case) has been changed; only then will this function be rendered.

```js
import React, { memo, useState } from "react";
import { useContext } from "react";
import { GrudgeContext } from "./GrudgeContext";
const NewGrudge = memo(({ onSubmit }) => {
  const { addGrudge } = React.useContext(GrudgeContext);
  const [person, setPerson] = useState("");
  const [reason, setReason] = useState("");

  const handleChange = (event) => {
    event.preventDefault();
    addGrudge({ person, reason });
  };

  return (
    <form className="NewGrudge" onSubmit={handleChange}>
      <input
        className="NewGrudge-input"
        placeholder="Person"
        type="text"
        value={person}
        onChange={(event) => setPerson(event.target.value)}
      />
      <input
        className="NewGrudge-input"
        placeholder="Reason"
        type="text"
        value={reason}
        onChange={(event) => setReason(event.target.value)}
      />
      <input className="NewGrudge-submit button" type="submit" />
    </form>
  );
});

export default NewGrudge;
```

### useCallback

useCallback is a React Hook that lets you cache a function definition between re-renders.

```js
const [grudges, dispatch] = useReducer(reducer, initialState);
const addGrudge = useCallback(
  ({ person, reason }) => {
    dispatch({
      type: GRUDGES_ADD,
      payload: {
        person,
        reason,
      },
    });
  },
  [dispatch]
);
```

### useRef;

useRef is a React Hook that lets you reference a value that’s not needed for rendering.
and create a mutable variable which will not re-render the components

useRef(initialValue) is a built-in React hook that accepts one argument as the initial value and returns a reference (aka ref). A reference is an object having a special property current.

```js
import { useRef } from "react";
function MyComponent() {
  const reference = useRef(initialValue);
  const someHandler = () => {
    // Access reference value:
    const value = reference.current;
    // Update reference value:
    reference.current = newValue;
  };
}
```

Updating a reference doesn't trigger a component re-rendering.

```js
import { useRef } from "react";
function LogButtonClicks() {
  const countRef = useRef(0);

  const handle = () => {
    countRef.current++;
    console.log(`Clicked ${countRef.current} times`);
  };
  console.log("I rendered!");
  return <button onClick={handle}>Click me</button>;
}
```

#### Accessing DOM elements

```js
import { useRef, useEffect } from "react";
function AccessingElement() {
  const elementRef = useRef();
  useEffect(() => {
    const divElement = elementRef.current;
    console.log(divElement); // logs <div>I'm an element</div>
  }, []);
  return <div ref={elementRef}>I'm an element</div>;
}
```

### useLayoutEffect

useEffect hook serves asynchronously, whereas the useLayoutEffect hook works synchronously;
![Example useLayoutEffect](https://miro.medium.com/max/828/1*GeEvUL0Zl3FVlSzNhFKiSg.gif)

```js

```
