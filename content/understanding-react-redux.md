---
title: Understanding React + Redux
image: /images/react-redux.jpg
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - ConsoleLog
date: Tue Aug 6 2019 09:13:51 GMT-0400 (Eastern Daylight Time)
tags:
  - new
---


Redux is an important part of the React ecosystem and is an important technology to learn if you want to become a React Developer.

To be completely transparent, it took me an embarrassing long time to really understand Redux. I understood the need and what it accomplished, but the complexity of it really through me off. The initial setup seemed daunting and unnecessary, but as I began to develop more complex apps with React, it was time to revisit Redux. I dedicated a good amount of time to simply building applications with it and the light bulb finally went off. In this post, I will break it down as much as possible to help your understanding of the technology.

##Why?

One of the main problems Redux solves is having the state of your entire application in one store. If we were to try to do this in React without Redux, 
we would run into the problem of "prop drilling", which is the act of passing props through components who do nothing with the props other than pass them to child components. That can quickly get messy and lead us down a rabbit hole when trying to make changes to our application. 

![](/images/flow.svg)

With redux, we can simply "connect" components to specific parts of the state in the store as needed.

##Terminology

![](/images/flow.png)
Redux comes with some new terminology that we have to become familiar with in order to understand what we're doing.

**Actions:** An action is a plain object that represents an intention to change the state. Actions are the only way to get data into the store. Ex:

```
{type: 'INCREMENT_COUNTER'}
```

**Action Creators:** An Action Creator is simply a function that returns an action.


Actions must have a type field that indicates the type of action being performed. Types can be defined as constants and imported from another module. Actions can also an optional payload field. Ex:

```
import { INCREMENT_COUNT} from './counterTypes';

const incrementCounter () => ({
  type: INCREMENT_COUNTER
})
```

**Dispatch:** A dispatch function is a function that accepts an action, which then dispatches a function to the store. Ex:
```
store.dispatch(incrementCounter()),
```

**Reducer:** A reducer is a function that accepts an accumulation and a value and returns a new accumulation. They are used to reduce a collection of values down to a single value. This will make more sense later.
```
import { INCREMENT_COUNT, DECREMENT_COUNT } from "./counterTypes";



const INITIAL_STATE = {
    count: 0
}

export const counterReducer = (state=INITIAL_STATE, action) => {
    switch(action.type){
        case INCREMENT_COUNT:
            return{
                ...state,
                count: state.count + 1
            }
        case DECREMENT_COUNT:
        return{
            ...state,
            count: state.count - 1
        }
        default: 
        return state
    }
}
```

**Store:** A store is an object that holds the application's state tree.

##Getting to the code

Whenever I learn something new, I need to know why this is even a thing. What problem does this new technology solve and does it solve problems that I currently face or will face in the future. With that in mind, we'll start by creating a simple React counter application without React. 

The easiest way to get started to to create a new application using create-react-app. We'll call the app counter-app.

```
npm install -g create-react-app
create-react-app counter-app
npm start
```

In the code block above, we installed the create-react-app package globally, created our counter application and spun up the dev server.

Within App.js, we're going to delete the functional component and add a class component with the following code:

```
import React from 'react';
import './App.scss'; 

class App extends React.Component{
  constructor(){
    super()
    this.state = {
      counter: 0
    }
  }

  increment = () => {
    this.setState(state =>{
      return {counter: state.counter + 1}
    });
  }

  decrement = () => {
    this.state.counter < 1 ? this.setState({counter: this.state.counter}): 
    this.setState(state =>{
      return {counter: state.counter - 1}
    });
  }
  render(){
    const {counter} = this.state;
    return(
      <div className="App">
        <h1>Counter</h1>
        <div className="counter-container">
          <button onClick={this.increment} className="button">+</button>
          <span>{counter}</span>
          <button onClick={this.decrement} className="button">-</button>
        </div>
      </div>
    )
  }
}

export default App;

```

Change App.css to App.scss (assumming you have sass installed) and copy the following code

```
 .App{  
     align-items: center;
     background: linear-gradient(to right, #8e2de2, #4a00e0);
     display: flex;
     flex-direction: column;
     height: 100vh;
     justify-content: center;

     h1, span{
         color: #fff;
     }
   .counter-container{
        display: flex;
        width: 400px;
        justify-content: space-around;
        align-items: center;

        .button{
            background: #fff;
            cursor: pointer;
            font-size: 1.5em;
            height: 50px;
            width: 100px;
        }   

        span{
            font-size: 2em;
        }
    }
 }
```

![](/images/counter.gif)

When we click the buttons, they'll call the respective increase or decrease functions, which will update the state and force a re-render of the number in the browser.   

#React-Redux

The first thing we'll do is add React and Redux to our project

```
npm install --save react react-redux
```

Next, in our src folder, we'll create a new folder called redux that will contain our redux files. Let's create a new file called counterTypes.js and copy the following code:

```
    export const INCREMENT_COUNT = 'INCREMENT_COUNT',
    export const DECREMENT_COUNT = 'DECREMENT_COUNT',
```

As a best practice, action types should typically defined as string constants. Next create a file called counterActions.js and copy the following code:

```
import { INCREMENT_COUNT, DECREMENT_COUNT } from './counterTypes';


export const incrementCounter = () => ({
    type: INCREMENT_COUNT
})

export const decrementCounter = () => ({
    type: DECREMENT_COUNT
})
```

Let's take a moment to breakdown the code above. First, we imported the two string constants we created in the counterTypes.js file. The two functions that we created are called Action Creators and the objects that they return are called Actions. To revisit the definition of an Action: An Action is a plain object that represents an intention to change the state. Actions are the only way to get data into the store. So, when these functions are called, the "Action" is to increment or decrement the counter.

Next create a file called counterReducer.js and copy the following code:

```
import { INCREMENT_COUNT, DECREMENT_COUNT } from "./counterTypes";



const INITIAL_STATE = {
    counter: 0
}

export const counterReducer = (state=INITIAL_STATE, action) => {
    switch(action.type){
        case INCREMENT_COUNT:
            return{
                ...state,
                counter: state.counter + 1
            }
        case DECREMENT_COUNT:
        return{
            ...state,
            counter: state.counter - 1
        }
        default: 
        return state
    }
}
```

In this code snippet, we start off by again importing our types string constants. We then create an INITIAL__STATE object. The first time our application is run, Redux will call our reducer with the INITIAL_STATE, so the INITIAL_STATE should be shaped the way you want your state to be displayed initially. In our case, we want our counter to start at 0;




What the reducer function does is take the current state and an action and returns a new state. It should return the initial state the first time it's called, which we called 'INITIAL_STATE' and set the counter property to zero;


Now let's create another new file called store.js and copy the following code:

```
import {createStore} from 'redux';
import {counterReducer} from './

export const store = createStore(countReducer);
```

Here we're creating our store. When our app is loaded initially, createStore will call our counterReducer, which will return the initial state, which is zero in our case. All other updates will be made when we dispatch our actions, which I will expalin later.

In index.js, let's import Provider from 'react-redux as well as our store.js from './redux/store.js;

```
import {Provider} from 'react-redux;
import store from './redux/store.js;
```

What the Provider component allows us to do is wrap our entire application with in it and pass down state to all child components. If a child component needs state, it can now "connect" to our store, which we will do shortly, but first let's wrap our <App/> component with Provider and pass in the store as a prop.

```
ReactDOM.render(<Provider store={store}><App /></Provider>, document.getElementById('root'));
```

Initially, we converted our App.js component into a class component in order to access state. Now we've going to switch back to a functional component to get rid of a lot of code that we don't need.

```
import React from 'react';
import './App.scss';
import {connect} from 'react-redux';
import { incrementCounter, decrementCounter } from './redux/counterActions';


const App = ({count, add, subtract}) => (
<div className="App">
        <h1>Counter</h1>
        <div className="counter-container">
          <button onClick={() => add()} className="button">+</button>
          <span>{count}</span>
          <button onClick={() => subtract()} className="button">-</button>
        </div>
      </div>
)


const mapStateToProps =(state)=>{
  return{count: state.count}
};

const mapDispatchToProps = (dispatch) => ({
  add: () => dispatch(incrementCounter()),
  subtract: () => dispatch(decrementCounter())
})

export default connect(mapStateToProps, mapDispatchToProps)(App);
```

This is where everything will finally come together. We start by importing connect from 'react-redux', this is what will allow our component to "listen" or "connect" to our store. We also imported our incrementCounter and decrementCounter action creators from our actions file.

If you take a look at the bottom of our file, you'll notice that we have the connect function and we're passing in mapStateToProps and mapDispatch to props. Let's take a closer look at what everything is doing.

##Connect
The Connect function is a higher-order function, which is essentially just a function that returns another function. We then call that function while passing in our App component. Essentially connect hooks into Redux, pulls out the entire state and passes it to your application through the mapStateToProps function.

##MapStateToProps
MapStateToProps returns the specific part of the state that your component needs and passes it into your component as props. In our case, we retrieve the count and pass it into our component as props. We then use the count prop to display the current number. In more complex applications, the state would be much larger and different components would connect to different parts of the state (or to the same part as other component). Our example is pretty simple, but you can see the overall purpose of mapStateToProps.

##MapDispatchToProps
The only way we can trigger a state change is by dispatching an action. In an React-Redux application, the mapDispatchToProps argument is responsible for enabling a component to dispatch actions. The mapDispatchToProps function is where we dispatch our actions.


![](/images/counter.gif)