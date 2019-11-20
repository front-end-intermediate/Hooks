# Hooks

- [Hooks](#hooks)
  - [useState](#usestate)
    - [Rules of Hooks](#rules-of-hooks)
    - [Update State Based on Previous State](#update-state-based-on-previous-state)
    - [State as an Array](#state-as-an-array)
    - [State as an Object](#state-as-an-object)
    - [Code Sandbox Exercises](#code-sandbox-exercises)
    - [State](#state)
    - [Props in State?](#props-in-state)
    - [Declarative Programming](#declarative-programming)
    - [Examples](#examples)
      - [Expanding/Collapsing an Accordion control](#expandingcollapsing-an-accordion-control)
      - [Opening and Closing a Dialog](#opening-and-closing-a-dialog)
  - [useRef and Forms](#useref-and-forms)
    - [Controlled Inputs](#controlled-inputs)
    - [Uncontrolled Inputs](#uncontrolled-inputs)
  - [useEffect](#useeffect)
    - [Limiting When an Effect Runs](#limiting-when-an-effect-runs)
    - [Focusing an Input Automatically](#focusing-an-input-automatically)
    - [Only Run on Mount and Unmount](#only-run-on-mount-and-unmount)
    - [Fetching Data with useEffect](#fetching-data-with-useeffect)
    - [Refetch When Data Changes](#refetch-when-data-changes)
    - [Making Visible DOM Changes](#making-visible-dom-changes)
    - [Exercises](#exercises)

## useState

Create a new project called "usestate" and start off with a blank index.js:

```sh
$ npx create-react-app usestate
$ cd usestate
$ rm src/*
$ touch src/index.js
```

```js
import React, { useState } from "react";
import ReactDOM from "react-dom";

class OneTimeButton extends React.Component {
  state = {
    clicked: false
  };
  handleClick = () => {
    // The handler won't be called if the button 
    // is disabled, so if we got here, it's safe 
    // to trigger the click.
    this.props.onClick();
    // Ok, no more clicking.
    this.setState({ clicked: true });
  };

  render() {
    return (
      <button onClick={this.handleClick} disabled={this.state.clicked}>
        You can only click once
      </button>
    );
  }
}

ReactDOM.render(
  <OneTimeButton onClick={() => alert("hi")} />,
  document.querySelector("#root")
);

```

Convert to a functional component:

```js
function OneTimeButtonFn({ onClick }) {
  const [clicked, setClicked] = React.useState(false);

  const handleClick = () => {
    onClick();
    // Ok, no more clicking.
    setClicked(true);
  };

  return (
    <button onClick={handleClick} disabled={clicked}>
      You Can Only Click Me Once
    </button>
  );
}
```

The useState hook takes the initial state as an argument (we passed false) and it returns an array with 2 elements: 

- the current state
- a function to change the state

Class components have one big state object, and a function `this.setState` to change the whole thing at once.

Function components come with no state at all, but the useState hook allows us to add little nuggets of state as we need them. So if all we need is a single boolean, we can create a bit of state to hold that.

Since we’re creating these pieces of state in a ad-hoc manner, and there’s no component-wide setState function, it makes sense that we’d need a function for updating each piece of state. So it’s a pair: one value, one function.

Each useState can store one value, and the value can be any JS type – a number, boolean, object, array, etc.

The bracket syntax `[clicked, setClicked] = ...` to the left of the equals sign is ES6 array destructuring - not  special hooks syntax. 

Array destructuring works similarly to object destructuring we’ve been using throughout the class, except that because array elements don’t have names, you assign names to the items when you destructure them.

With useState it’s common to name the returned values e.g. `foo` and `setFoo`, but you can call them whatever you like. The first element is the current value, and the second element is a setter function.

### Rules of Hooks

1. Only call hooks at the top level of your function. Don’t put them in loops, conditionals, or nested functions. In order for React to keep track of your hooks, the same ones need to be called in the same order every single time. If you called useState from inside an if, for instance, and it ran during the first render but got skipped during the second, React would be very confused.
2. Only call hooks from React function components or from custom hooks. Don’t call them from outside a component. Keeping all the calls inside components and custom hooks makes your code easier to follow because all the related logic is grouped together.
3. The names of custom hooks must start with "use". 

### Update State Based on Previous State

Build a step tracker. Every time you take a step, click the button. At the end of the day, it will tell you how many steps you took. 

```js
function StepTracker() {
  const [steps, setSteps] = useState(0);

  function increment() {
    setSteps(steps => steps + 1);
  }

  return (
    <div>
      Today you've taken {steps} steps!
      <br />
      <button onClick={increment}>I took another step</button>
    </div>
  );
}
```

We've extracted the increment function, instead of inlining the arrow function on the button’s onClick prop.

### State as an Array

Build a random number generator. Clicking the button adds a new random number to the UI list:

```js
function RandomList() {
  const [items, setItems] = useState([]);

  const addItem = () => {
    setItems([
      ...items,
      {
        id: items.length,
        value: Math.random() * 100
      }
    ]);
  };

  return (
    <>
      <button onClick={addItem}>Add a number</button>
      <ul>
        {items.map(item => (
          <li key={item.id}>{item.value}</li>
        ))}
      </ul>
    </>
  );
}
```

Note: we’re initializing the state to an empty array [], and note the addItem function. 

The state updater function (here setItems) doesn’t "merge" new values with old – it overwrites the state with the new value. 

In order to add an item to the array, we’re using the ES6 spread operator `...` to copy the existing items into the new array, and inserting the new item at the end.

### State as an Object

Since the setter function returned by useState will overwrite the state each time you call it, it works differently from the class-based `this.setState`.

`this.setState` would shallow-merge the object you passed it into the existing state, taking care not to clobber the other state already in there.

The useState setter, instead, will clobber everything. It replaces the entire value with whatever you pass in. 

Build an example where state is an object with a couple values:

```js
const MultiCounter = () => {
  const [counts, setCounts] = useState({
    countA: 0,
    countB: 0
  });

  const incA = () =>
    setCounts(counts => ({
      ...counts,
      countA: counts.countA + 1
    }));

  const incB = () =>
    setCounts(counts => ({
      ...counts,
      countB: counts.countB + 1
    }));

  const badIncA = () =>
    setCounts({
      countA: counts.countA + 1
    });

  return (
    <>
      <div>A: {counts.countA}</div>
      <div>B: {counts.countB}</div>
      <button onClick={incA}>Increment A</button>
      <button onClick={incB}>Increment B</button>
      <button onClick={badIncA}>Increment A Badly</button>
    </>
  );
};
```

If your state is a complex value like an object or array, you need to take care, when updating it, to copy in all the other parts that you don’t intend to change. 

The `...` spread operator is the principal means for making copies of arrays and objects.

### Code Sandbox Exercises

1. Create a Room component with a "lightswitch" button and some text describing "The room is lit" or "The room is dark". Clicking the button should toggle the light on and off, and update the text. Use the useState hook to store the lightswitch state.

```css
html,
body,
#root,
.room {
  height: 100vh;
  margin: 0;
  text-align: center;
  font-family: "Georgia", serif;
  font-size: 1.5rem;
}

button {
  padding: 0.25rem 0.5rem;
  font-size: 1rem;
}

.room {
  padding: 30px;
}

.lit {
  background-color: white;
  color: black;
}

.dark {
  background-color: black;
  color: white;
}

```

```js
import React, {useState} from "react";
import ReactDOM from "react-dom";

import "./styles.css";

const Room = () => {
  const [isLightOn, setLight] = useState(true);

  const lightedness = isLightOn ? "lit" : "dark";

  const flipLight = () => {
    setLight(!isLightOn);
  };

  return (
    <div className={`room ${lightedness}`}>
      the room is {lightedness}
      <br />
      <button onClick={flipLight}>flip</button>
    </div>
  );
};

const rootElement = document.getElementById("root");
ReactDOM.render(<Room />, rootElement);

```

2. Create a RandomList component that shows a button, and a list of random numbers. When you click the button, add another random number to the list. Store the array of numbers with useState. The initial state should be an empty array.

```js
import React, { useState } from "react";
import ReactDOM from "react-dom";
import "./index.css";

const RandomList = () => {
  const [numbers, setNumbers] = useState([]);

  const addNumber = () => {
    // The "updater" method (passing a function that receives the old state and returns the new one) is
    // the safest choice when the new state depends on the old state. It guarantees that the old state
    // won't be stale.
    setNumbers(nums => [...nums, Math.random()]);

    // In this example, this would work too. 
    // setNumbers([...numbers, Math.random()]);

    /* This won't work, because pushing onto an array doesn't replace the original array,
       and React won't re-render unless the state value looks new. Test it: */
    // numbers.push(Math.random());
    // setNumbers(numbers);
  };

  return (
    <div>
      <h1>Random Numbers</h1>
      <button onClick={addNumber}>Add a Number</button>
      <ul>
        {numbers.map((number, index) => (
          <li key={index}>{number}</li>
        ))}
      </ul>
    </div>
  );
};

ReactDOM.render(<RandomList />, document.querySelector("#root"));

```

3. Create a component called AudioControls with 4 pieces of state: "volume", "bass", "mid, and"treble", each storing a value between 1 and 100. 

```css
* {
  font-family: Helvetica, arial, sans-serif;
}
.audio-controls {
  margin-bottom: 3em;
}
.control {
  display: flex;
  max-width: 200px;
  margin-bottom: 20px;
}
.control > div {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;
}
.control .value {
  font-size: 2em;
  margin-bottom: 3px;
}
.control .label {
  font-size: 0.9em;
  opacity: 0.7;
}
.control button {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  border: 2px solid #ccc;
  font-size: 20px;
}

```

```js
import React, { useState } from "react";
import ReactDOM from "react-dom";
import "./index.css";

const Control = ({ value, children, onIncrease, onDecrease }) => (
  <div className="control">
    <button onClick={onDecrease}>&ndash;</button>
    <div>
      <span className="value">{value}</span>
      <span className="label">{children}</span>
    </div>
    <button onClick={onIncrease}>+</button>
  </div>
);

// This version of the component stores its state as 4 separate variables
const AudioControlsWithMultipleVariables = () => {
  const [volume, setVolume] = useState(47);
  const [treble, setTreble] = useState(13);
  const [mid, setMid] = useState(32);
  const [bass, setBass] = useState(50);

  return (
    <div className="audio-controls">
      <Control
        value={volume}
        onIncrease={() => setVolume(volume + 1)}
        onDecrease={() => setVolume(volume - 1)}
      >
        VOLUME
      </Control>

      <Control
        value={treble}
        onIncrease={() => setTreble(treble + 1)}
        onDecrease={() => setTreble(treble - 1)}
      >
        TREBLE
      </Control>

      <Control
        value={mid}
        onIncrease={() => setMid(mid + 1)}
        onDecrease={() => setMid(mid - 1)}
      >
        MID
      </Control>

      <Control
        value={bass}
        onIncrease={() => setBass(bass + 1)}
        onDecrease={() => setBass(bass - 1)}
      >
        BASS
      </Control>
    </div>
  );
};

// This version stores the state in a single object
const AudioControlsWithOneObject = () => {
  const [{ volume, bass, mid, treble }, setValues] = useState({
    volume: 53,
    bass: 17,
    mid: 51,
    treble: 32
  });

  const increase = key => () => {
    setValues(values => ({
      ...values,
      [key]: values[key] + 1
    }));
  };

  const decrease = key => () => {
    setValues(values => ({
      ...values,
      [key]: values[key] - 1
    }));
  };

  return (
    <div className="audio-controls">
      <Control
        value={volume}
        onIncrease={increase("volume")}
        onDecrease={decrease("volume")}
      >
        VOLUME
      </Control>

      <Control
        value={treble}
        onIncrease={increase("treble")}
        onDecrease={decrease("treble")}
      >
        TREBLE
      </Control>

      <Control
        value={mid}
        onIncrease={increase("mid")}
        onDecrease={decrease("mid")}
      >
        MID
      </Control>

      <Control
        value={bass}
        onIncrease={increase("bass")}
        onDecrease={decrease("bass")}
      >
        BASS
      </Control>
    </div>
  );
};

ReactDOM.render(
  <>
    <h1>With Multiple Variables</h1>
    <AudioControlsWithMultipleVariables />

    <h1>With A Single Object</h1>
    <AudioControlsWithOneObject />
  </>,
  document.querySelector("#root")
);
```

### State

As a general rule, data that is stored in state should be referenced inside render. Component state is for storing UI state – things that affect the visual rendering of the page. This makes sense because any time state is updated, the component will re-render.

If modifying a piece of data does not visually change the component, that data shouldn’t go into state. 

Some things that make sense to put in state:

- User-entered input (text boxes and other form fields)
- Current or selected item (the current tab, the selected row)
- Data from the server (a list of recipes, the number of "likes") 
- Open/closed state (modal open/closed, sidebar expanded/hidden)

### Props in State?

Avoid copying props into state. It creates a second source of truth for your data, which usually leads to bugs. If you ever find yourself copying a prop into state and then thinking, "Now how am I going to keep this updated?" – take a step back and rethink.

Components automatically re-render when their parents do, and receive fresh props each time, so there’s no need to duplicate the props into state and then try to keep it up to date.

### Declarative Programming

If you came from a framework or language where you primarily call functions to make things happen in a certain order - imperative programming - you need to adjust your mental model in order to work effectively with React. 

### Examples

#### Expanding/Collapsing an Accordion control

_The old way_: Clicking a toggle button opens or closes the accordion by calling its toggle function.
The Accordion knows whether it is open or closed.

The declarative way: The Accordion can be displayed in either a 'open' or 'closed' state, and we store that information as a flag inside the parent component’s state (not inside the Accordion). We tell the Accordion which way to render by passing isOpen as a prop. 

When isOpen is true, it renders as open. When isOpen is false, it renders as closed, e.g.:

```js
<Accordion isOpen={true}/> 
// or
<Accordion isOpen={false}/>
```

#### Opening and Closing a Dialog

_The old way_: Clicking a button opens the modal. Clicking its Close button closes it.

The declarative way: Whether or not the Modal is open is a state. It’s either in the "open" state or the "closed" state. So, if it’s "open", we render the Modal. If it’s "closed" we don’t render the modal. Moreover, we can pass an onClose callback to the Modal – this way the parent component gets to decide what happens when the user clicks Close.

```js
<div> 
  {this.state.isModalOpen &&
  <Modal onClose={this.handleClose}/>} 
</div>
```

Whenever you can, it’s best to keep components stateless. Components without state are easier to write, and easier to reason about. Sometimes this isn’t possible, but often, pieces of data you initially think should go into internal state can actually be lifted up to the parent component, or higher.

## useRef and Forms

### Controlled Inputs

With controlled inputs you are responsible for controlling state, need to pass in a value, and keep that value updated as the user types:

```js
import React, { useState } from "react";
import ReactDOM from "react-dom";

const ControlledInputFn = () => {
  const [text, setText] = useState("");

  const handleChange = event => {
    setText(event.target.value);
    console.log(text)
  };

  return <input type="text" value={text} onChange={handleChange} />;
};

const rootElement = document.getElementById("root");
ReactDOM.render(<ControlledInputFn />, rootElement);

```

We create a piece of state to hold the input’s value and store that in text. 

Every time you press a key, the handleChange function will get called with the input’s current value (the whole thing, not just the most recent key).

Note: the `value prop` tells the input what to display.

_Try_ removing or commenting out the call to setText, then try to type in the box. Nothing happens because the value is stuck at the initial value.

Try changing setText to ignore the event data, and instead set the text to the same value:

```js
const TrickInput = () => {
  const [text, setText] = useState("try typing something");

  const handleChange = event => {
    setText("haha nope");
  };

  return <input type="text" value={text} onChange={handleChange} />;
};
```

This technique is useful if you need custom validation or formatting because you can do both in the handleChange function. 

Don’t want the user to type numbers? Strip out the numbers using `replace(<regex>)` before updating the state:

```js
const NoNumbersInput = () => {
  const [text, setText] = useState("");

  const handleChange = event => {
    let text = event.target.value;
    setText(text.replace(/[0-9]/g, ""));
  };

  return <input type="text" value={text} onChange={handleChange} />;
};
```

### Uncontrolled Inputs

When an input is uncontrolled, it manages its own internal state. 

```js
const EasyInput = () => <input type="text" />;
```

When you want to get a value out of it you use a ref. 

A ref gives access to the input’s underlying DOM node so you can access its value.

In function components we can call the useRef hook to create an empty ref, and then pass that into a ref prop on the input.

```js
const RefInput = () => {
  const input = useRef();

  const showValue = () => {
    alert(`Input contains: ${input.current.value}`);
  };

  return (
    <div>
      <input type="text" ref={input} />
      <button onClick={showValue}>Alert the Value!</button>
    </div>
  );
};
```

## useEffect

Adding lifecycle methods like `componentDidMount` to class components allow you to make things happen at specific times – say, after a component mounts, or after it re-renders - `componentDidUpdate`.

The generic name for these actions is "side effects" - a term that comes from functional programming. 

In the ideal world, you could implement your UI as a pure function of props and state: "given a specific set of state like this, the app should look like that." That’s the core idea behind React, and it works well most of the time.

Sometimes though, you need to do something that doesn’t fit within that box - making a request to fetch data, or focusing an input control when the page loads. Basically, anything that doesn’t fit the paradigm of stateful updates can be handled by a side effect.

With the useEffect hook, you can respond to lifecycle events directly inside function components. Namely, three of them: `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` - all with one function.

Remember that every change to props or state will cause a component to re-render. On every render, useEffect will have a chance to run. 

By default, your effects will execute on every render, but we can limit how often they run.

Youc an think of useEffect as "if-this-then-that" for your React components.

Let’s create a project where we can play around with useEffect. Create an empty project and open up the index.js file.

```sh
$ npx create-react-app useeffect-hook 
$ cd useeffect-hook
$ rm src/*
$ touch src/index.js
```

```js
import React, { useState, useEffect } from "react";
import ReactDOM from "react-dom";

const LogEffect = () => {
  const [text, setText] = useState("");

  useEffect(() => {
    console.log("latest value: ", text);
  });

  return <input value={text} onChange={e => setText(e.target.value)} />;
};

ReactDOM.render(<LogEffect />, document.querySelector("#root"));
```

Start up the example and open the browser's console. 

Note the app logged "latest value:" even though you haven’t typed anything into the box. 

`useEffect` always runs after the initial render and there’s no way to turn that off - similar to componentDidMount.

Now try typing in the box, and you’ll see that it logs a message for every character you type. That’s because the effect is running on (after) every render.

### Limiting When an Effect Runs

Often, you’ll only want an effect to run in response to a specific change - when a prop’s value changes or a change occurs to state. That’s what the second argument of useEffect is for: it’s a list of dependencies.

Example: "If the blogPostId prop changes, then download that blog post and display it":

```js
useEffect(() => { 
  fetch(`/posts/${blogPostId}`)
  .then(content => setContent(content))
}, [blogPostId])
```

### Focusing an Input Automatically

Focus an input control upon first render, using useEffect combined with the useRef hook.

```js
import React, { useEffect, useState, useRef } from "react";
import ReactDOM from "react-dom";

function App() {
  // Store a reference to the input's DOM node
  const inputRef = useRef();
  // Store the input's value in state
  const [value, setValue] = useState("");

  useEffect(() => {
    console.log("render");
    inputRef.current.focus();
  }, [inputRef]);

  return (
    <input
      ref={inputRef}
      value={value}
      onChange={e => setValue(e.target.value)}
    />
  );
}
ReactDOM.render(<App />, document.querySelector("#root"));

```

We’re creating an empty ref with useRef. Passing it to the input’s ref prop takes care of setting it up once the DOM is rendered and, importantly, the value returned by useRef will be stable between renders – it won’t change.

So, even though we’re passing [inputRef] as the 2nd argument of useEffect, it will effectively only run once, right after the component is mounted. This is basically `componentDidMount`.

Note: the input is auto-focused upon page load. 

Try typing in the box. Each character triggers a re-render, but if you look at the console, you’ll see that "render" is only printed once. The useEffect is looking at the value of inputRef each time, seeing that it hasn’t changed since the previous render, and then deciding not to run your effect function.

Here’s another way to think of this dependency array: it should contain every variable that the effect function uses from the surrounding scope. If it uses a prop or a piece of state, that goes in the array. 

### Only Run on Mount and Unmount

If you pass no array you’re telling useEffect to run every render.

An empty array says "this effect depends on nothing," and so it will only run once, after the first render.

```js
import React, { useEffect, useState, useRef } from "react";
import ReactDOM from "react-dom";

function App() {
  // Store a reference to the input's DOM node
  const inputRef = useRef();
  // Store the input's value in state
  const [value, setValue] = useState("");

  useEffect(() => {
    console.log("mounted");
    return () => console.log("unmounting...");
  }, []); // <-- add an empty array here

  return (
    <input
      ref={inputRef}
      value={value}
      onChange={e => setValue(e.target.value)}
    />
  );
}
ReactDOM.render(<App />, document.querySelector("#root"));

```

Note: we’re returning a function from the effect, and that function will be called to clean up the effect.

Effects that:

- start a timer or interval
- start a request that needs to be cancelled
- add an event listener that needs to be removed 
  
are all examples of occasions when you’ll want to return the cleanup function.

Note: passing an empty array is prone to bugs. It’s easy to forget to add an item to it if you add a dependency, and if you miss a dependency, then that value will be stale the next time useEffect runs and might cause some strange problems.

 Make sure that when you use an empty array [], you really mean it.

 ### Fetching Data with useEffect

 In a class component, you’d put this code in the `componentDidMount` method. To do it with hooks use useEffect. Typically you’ll also need useState to store the data.

 Examine the json returned from the Reddit API:

 `https://www.reddit.com/r/reactjs.json`

Here's a component that fetches posts from Reddit and displays them (note: danger zone).

Key this in:

```js
import React, { useEffect, useState, useRef } from "react";
import ReactDOM from "react-dom";

function Reddit() {
  const [posts, setPosts] = useState([]);
  useEffect(() => {
    fetch("https://www.reddit.com/r/reactjs.json")
      .then(res => res.json())
      .then(json => setPosts(json.data.children.map(c => c.data)));
    console.log("ran");
  }); 
  // we didn't pass the 2nd arg.

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

ReactDOM.render(<Reddit />, document.querySelector("#root"));

```

Passing no 2nd argument causes the useEffect to run every render. Then, when it runs, it fetches the data and later updates the state. Once the state is updated, the component re-renders, which triggers the useEffect again. 

### Refetch When Data Changes

Expand the example to cover another common problem: how to re-fetch data when something changes, in this case, the name of the subreddit.

Change the Reddit component to accept the subreddit as a prop, fetch the data based on that subreddit, and only re-run the effect when the prop changes:

```js
import React, { useEffect, useState, useRef } from "react";
import ReactDOM from "react-dom";

function Reddit({ subreddit }) {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    fetch(`https://www.reddit.com/r/${subreddit}.json`)
      .then(res => res.json())
      .then(json => setPosts(json.data.children.map(c => c.data)));
    console.log("ran");
  }, [subreddit, setPosts]);

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

ReactDOM.render(
  <Reddit subreddit="reactjs" />,
  document.querySelector("#root")
);

```

This is hard-coded, but we will customize it by wrapping the Reddit component with one that lets us change the subreddit. 

Add a new App component, and render it:

```js
import React, { useEffect, useState, useRef } from "react";
import ReactDOM from "react-dom";

function Reddit({ subreddit }) {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    fetch(`https://www.reddit.com/r/${subreddit}.json`)
      .then(res => res.json())
      .then(json => setPosts(json.data.children.map(c => c.data)));
    console.log("ran");
  }, [subreddit, setPosts]);

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

function App() {
  const [inputValue, setValue] = useState("reactjs");
  const [subreddit, setSubreddit] = useState(inputValue);
  const handleSubmit = e => {
    e.preventDefault();
    setSubreddit(inputValue);
  };

  return (
    <>
      <form onSubmit={handleSubmit}>
        <input value={inputValue} onChange={e => setValue(e.target.value)} />
      </form>
      <Reddit subreddit={subreddit} />
    </>
  );
}

ReactDOM.render(<App />, document.querySelector("#root"));

```

The App component is keeping 2 pieces of state – the current input value, and the current subreddit. 

Submitting the input 'commits' the subreddit, which causes Reddit to re-fetch the data from the new selection.

Note: wrapping the input in a form allows the user to press Enter to submit.

Note: there’s no error handling. If you type a subreddit that doesn’t exist, the app will fail.

We could’ve used a single piece of state here to store the input, and send the same value down to the Reddit component – but then the Reddit component would be fetching data with every keypress.

Note the useState at the top, especially the second line:

```js
const [inputValue, setValue] = useState("reactjs"); 
const [subreddit, setSubreddit] = useState(inputValue);
```

We’re passing an initial value of "reactjs" to the first piece of state, and that makes sense. That value will never change.

But what about that second line? What if the initial state changes? (and it will, when you type in the box)

`useState` only uses the initial state once, the first time it renders. After that it’s ignored. So it’s safe to pass a transient value, like a prop that might change or some other variable.

### Making Visible DOM Changes

The useEffect function is the swiss army knife of hooks. It can be used for many purposes

- setting up subscriptions 
- creating and cleaning up timers 
- changing the value of a ref

Do not use it for making DOM changes that are visible to the user. 

The way the timing works, an effect function will only fire after the browser is done with layout and paint – too late, if you wanted to make a visual change.

For those cases, React provides the `useLayoutEffect` hook. It works the same as useEffect in terms of the arguments it takes. The difference is that it runs at the same time as componentDidMount would have – between when browser has updated the DOM and before those changes are painted to the screen.

Most of the time, useEffect is the one you want. But if your effect needs to measure DOM elements or change them in some visible way, then use a useLayoutEffect.

```js
const Demo = () => (
  <>
    <h2>Logging Example</h2>
    <LogEffect />
    <h2>Reddit Example</h2>
    <RedditInput />
  </>
);

ReactDOM.render(<Demo />, document.querySelector("#root"));
```

### Exercises

(Note: do not do these in CodeSandbox.)

1. Document Title: Render an input box and store its value with useState. Then set the document.title in an effect, keeping the page’s title in sync with the input.

```js
import React, { useState, useEffect } from "react";
import ReactDOM from "react-dom";

const App = () => {
  // Initialize the title from the document's title
  const [title, setTitle] = useState(document.title);

  useEffect(() => {
    // Since we're modifying something outside the
    // component, it needs to be done inside a useEffect.
    //
    // So here we set the title, and the `title` state is
    // guaranteed to be the most recent value.
    document.title = title;
  });

  return (
    <label>
      Enter a new title:
      <input
        value={title}
        /* 
          just setting state here, not changing the
          document.title directly!
        */
        onChange={e => setTitle(e.target.value)}
      />
    </label>
  );
};

ReactDOM.render(<App />, document.querySelector("#root"));

```


2. Add a event listener to the document and print a message every time the user clicks. (Note the clean up handler.)

```js
import React, { useEffect } from "react";
import ReactDOM from "react-dom";

const App = () => {
  useEffect(() => {
    const announceClick = e => console.log("clicked!", e.clientX, e.clientY);
    // Set up a listener for the click event.
    // We're passing a function here so that we can pass
    // the same function to removeEventListener and clean
    // it up.
    document.addEventListener("click", announceClick);
    return () =>
      // When the effect is cleaned up, remove the click handler
      document.removeEventListener("click", announceClick);
  }, []); // only run once, on mount

  return <div>click anywhere!</div>;
};

ReactDOM.render(<App />, document.querySelector("#root"));

```


1. The Reddit example is lacking error handling, and if you enter an invalid subreddit name, the app will break. Add code to intercept errors, handle them gracefully, and display an error message.

```js
import React, { useState, useEffect } from "react";
import ReactDOM from "react-dom";

// 1. Destructure the `subreddit` from props:
function Reddit({ subreddit }) {
  const [posts, setPosts] = useState([]);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Clear the error & data before fetching new data
    setError(null);
    setPosts([]);

    // Fetch posts
    fetch(`https://www.reddit.com/r/${subreddit}.json`)
      .then(res => {
        if (res.ok) {
          return res;
        }
        throw new Error("Could not fetch posts");
      })
      .then(res => res.json())
      .then(json =>
        // Save the posts into state
        setPosts(json.data.children.map(c => c.data))
      )
      .catch(error => {
        // Save the error in state
        setError(error.message);
      });
  }, [subreddit, setPosts]);

  return (
    <ul>
      {error ? error : posts.map(post => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}

function App() {
  // 2 pieces of state: one to hold the input value,
  // another to hold the current subreddit.
  const [inputValue, setValue] = useState("reactjs");
  const [subreddit, setSubreddit] = useState(inputValue);

  // Update the subreddit when the user presses enter
  const handleSubmit = e => {
    e.preventDefault();
    setSubreddit(inputValue);
  };

  return (
    <>
      <form onSubmit={handleSubmit}>
        <input value={inputValue} onChange={e => setValue(e.target.value)} />
      </form>
      <Reddit subreddit={subreddit} />
    </>
  );
}

ReactDOM.render(<App />, document.querySelector("#root"));

```


Note this portion:

```js
.then(res => {
  if (res.ok) {
    return res;
  }
  throw new Error("Could not fetch posts");
})
```

`fetch` has an `ok` property on the response.

If the response.ok property is true, we return the response.json(). If not, we throw an error.

You can also return a rejected Promise object, passing in the response, to trigger the catch() method.

```js
fetch('https://jsonplaceholder.typicode.com/postses').then(function (response) {
	// The API call was successful
	if (response.ok) {
		return response.json();
	} else {
		return Promise.reject(response);
	}
}).then(function (data) {
	// This is the JSON from our response
	console.log(data);
}).catch(function (err) {
	// There was an error
	console.warn('Something went wrong.', err);
});
```