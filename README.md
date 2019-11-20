# useState

Create a new project called “usestate” and start off with a blank index.js:

```sh
create-react-app usestate
cd usestate
rm src/*
touch src/index.js
```

```js
import React, { useState } from "react";
import ReactDOM from "react-dom";

class OneTimeButton extends React.Component {
  state = {
    clicked: false
  };
  handleClick = () => {
    // The handler won't be called if the button // is disabled, so if we got here, it's safe // to trigger the click.
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

useState is a hook because its name starts with “use” (that’s one of the Rules of Hooks – their names must start with “use”).

The useState hook takes the initial state as an argument (we passed false) and it returns an array with 2 elements: the current state, and a function to change the state.

Class components have one big state object, and a function this.setState to change the whole thing at once (plus it shallow-merges the new value).

Function components come with no state at all, but the useState hook allows us to add little nuggets of state as we need them. So if all we need is a single boolean, we can create a bit of state to hold that.

Since we’re creating these pieces of state in a sort of ad-hoc way, and there’s no component-wide setState function, it makes sense that we’d need a function for updating each piece of state. So it’s a pair: one value, one function.

Each useState can store one value, and the value can be any JS type – a number, boolean, object, array, etc.

The bracket syntax [clicked, setClicked] = ... to the left of the equal sign is array destructuring, and that’s built in to JavaScript since ES6 (it’s not a special hooks thing). It works similarly to object destructuring we’ve been using throughout the book, except that because array elements don’t have names, you get to assign names to the items when you destructure them.

With useState it’s common to name the returned values like foo and setFoo, but you can call them whatever you like. The first element is the current value, and the second element is a setter function.

Now I bet you have a lot of questions. Things like...

- How does React know what the old state was? When the component re-renders... won’t the state get re-created every time?
- How can I store more complex state? I have to keep track of more than one value.
- Why do hook names have to start with “use”? 
- If there’s a rule about naming... does that mean I can make my own hooks?


1. Only call hooks at the top level of your function. Don’t put them in loops, conditionals, or nested functions. In order for React to keep track of your hooks, the same ones need to be called in the same order every single time. If you called useState from inside an if, for instance, and it ran during the first render but got skipped during the second, React would be very confused.
2. Only call hooks from React function components, or from custom hooks (we’ll learn about those later). Don’t call them from outside a component (what would that even do?). Keeping all the calls inside components and custom hooks makes your code easier to follow too, because all the related logic is grouped together.
3. The names of custom hooks must start with “use”. 

## Update State Based on Previous State

We’ll build a, uh, “step tracker.” Every time you take a step, simply click the button. At the end of the day, it will tell you how many steps you took. 

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

## State as an Array

Here’s an example of a list of random numbers. Clicking the button adds a new random number to the list:

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
      <button onClick={addItem}>Add a number</button>{" "}
      <ul>
        {items.map(item => (
          <li key={item.id}>{item.value}</li>
        ))}
      </ul>{" "}
    </>
  );
}
```

Note: we’re initializing the state to an empty array [], and take a look at the addItem function. 

The state updater function (setItems, here) doesn’t “merge” new values with old – it overwrites the state with the new value. This is a departure from the way this.setState worked in classes.

In order to add an item to the array, we’re using the ES6 spread operator ... to copy the existing items into the new array, and inserting the new item at the end.

## State as an Object

Since the setter function returned by useState will overwrite the state each time you call it, it works differently from the class-based this.setState.

`this.setState` would shallow-merge the object you passed it into the existing state, taking care not to clobber the other stuff in there.

The useState setter, instead, will clobber everything. It replaces the entire value with whatever you pass in. Here’s an example where state is an object with a couple values:

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

The ... spread operator is a big help for making copies of arrays and objects.

## Exercises

1. Create a Room component with a “lightswitch” button and some text describing“The room is lit” or “The room is dark”. Clicking the button should toggle the light on and off, and update the text. Use the useState hook to store the lightswitch state.

```css
html,
body,
#root,
.room {
  height: 100%;
  margin: 0;
  text-align: center;
  font-family: 'Georgia', serif;
  font-size: 20px;
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
import React, { useState } from "react";
import ReactDOM from "react-dom";
import "./index.css";

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

ReactDOM.render(<Room />, document.getElementById("root"));

```

2. Create a RandomList component that shows a button, and a list of random numbers. When you click the button, add another random number to the list. Store the array of numbers with useState. The initial state should be an empty array.

```js
import React, { useState } from "react";
import ReactDOM from "react-dom";
import "./index.css";

const RandomList = () => {
  const [numbers, setNumbers] = useState([]);

  const addNumber = () => {
    // The "updater" form (passing a function that receives the old state and returns the new one) is
    // the safest choice when the new state depends on the old state. It guarantees that the old state
    // won't be stale (which can happen if you're accessing old state from inside a closure).
    setNumbers(nums => [...nums, Math.random()]);

    /* In this example, this would work too. Why won't "numbers" be stale, here, even though addNumber is a closure? */
    // setNumbers([...numbers, Math.random()]);

    /* This won't work, because pushing onto an array doesn't replace the original array,
       and React won't re-render unless the state value looks new. Try it out: */
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

3. Create a component called AudioControls with 4 pieces of state: “volume”, “bass”, “mid, and”treble”, each storing a value between 1 and 100. 

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

## State

How do you decide what should go into state?

As a general rule, data that is stored in state should be referenced inside render somewhere. Component state is for storing UI state – things that affect the visual rendering of the page. This makes sense because any time state is updated, the component will re-render.

If modifying a piece of data does not visually change the component, that data shouldn’t go into state. 

Some things that make sense to put in state:

- User-entered input (text boxes and other form fields)
- Current or selected item (the current tab, the selected row)
- Data from the server (a list of recipes, the number of “likes”) 
- Open/closed state (modal open/closed, sidebar expanded/hidden)

## Should Props Go in State?


You should avoid copying props into state. It creates a second source of truth for your data, which usually leads to bugs. If you ever find yourself copying a prop into state and then thinking, “Now how am I going to keep this updated?” – take a step back and rethink.


Components automatically re-render when their parents do, and receive fresh props each time, so there’s no need to duplicate the props into state and then try to keep it up to date.

## Declarative Programming

If you came from a framework or language where you primarily call functions to make things happen in a certain order (“imperative programming”), you need to adjust your mental model in order to work effectively with React. You’ll adjust pretty quickly with practice – you just need a few new examples or “patterns”

For example:

### Expanding/Collapsing an Accordion control

The old way: Clicking a toggle button opens or closes the accordion by calling its toggle function.
The Accordion knows whether it is open or closed.

The declarative way: The Accordion can be displayed in either the “open” state, or the “closed” state, and we store that information as a flag inside the parent component’s state (not inside the Accordion). We tell the Accordion which way to render by passing isOpen as a prop. When isOpen is true, it renders as open. When isOpen is false, it renders as closed.

```js
<Accordion isOpen={true}/> // or
<Accordion isOpen={false}/>
```

### Opening and Closing a Dialog

The old way: Clicking a button opens the modal. Clicking its Close button closes it.

The declarative way: Whether or not the Modal is open is a state. It’s either in the “open” state or the “closed” state. So, if it’s “open”, we render the Modal. If it’s “closed” we don’t render the modal. Moreover, we can pass an onClose callback to the Modal – this way the parent component gets to decide what happens when the user clicks Close.

```js
<div> 
  {this.state.isModalOpen &&
  <Modal onClose={this.handleClose}/>} 
</div>
```

Whenever you can, it’s best to keep components stateless. Components without state are easier to write, and easier to reason about. Sometimes this isn’t possible, but often, pieces of data you initially think should go into internal state can actually be lifted up to the parent component, or even higher.


