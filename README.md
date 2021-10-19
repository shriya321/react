# react
# How to use React Context API with Functional | Class Components
Consider a case where some data needs to be accessed by many components at different nested levels. In using <a href="https://reactjs.org/">React</a>, we have to pass data as props to the component tree so that whichever component needs the data can access it. By doing this, we are passing data from parent components to nested child components.

 

Getting data from component A to component Z by passing it to multiple layers of components. As Data has to be passed through multiple components, this problem is called Props drilling. Because some components only just get the props and passing it to the child component as props.  Below is an example of the above scenario.

# React Context
 

# Level 1 – Parent Component  (App.js)
 

import React, { Component } from "react";

import ReactDOM from "react-dom";

import UserProfile from "./UserProfile.js";


class App extends Component {

 constructor(props) {

   super(props);

   this.state = {

     currentUser: ""

   };

 }


 render() {

   return (

     <div>

       <UserProfile currentUser={this.state.currentUser} />

     </div>

   );

 }

}


const rootElement = document.getElementById("root");

ReactDOM.render(<App />, rootElement);
 

# Level – 2 UserProfile.js
 

import React from "react";

import CurrentUserProfile from "./CurrentUserProfile.js";


const UserProfile = props => {

 return (

   <div>

     <CurrentUserProfile currentUser={props.currentUser} />

   </div>

 );

};


export default UserProfile;
 

# Level – 3 CurrentUserProfile.js
 

import React from "react";


const CurrentUserProfile = props => {

 return (

   <div>

     <div>{props.currentUser}</div>

   </div>

 );

};


export default CurrentUserProfile;
 

In the above example, we are using 3 levels of nested components. We are passing currentUser information as props to the nested component. The UserProfile component is getting the data as props and passing it to the CurrentUserProfile component.

 

The UserProfile component is not using that props data in itself. Now consider a case where we have more than 3 levels of nested components, in this case, many components just only passing the props to their child components. This situation is really painful in large applications. 

 

Here comes React Context API into the Picture. Context provides a way to share values like these between components without having to explicitly pass a prop through every level of the tree. Context provides data that can be considered global for a tree of React components without explicitly passing data to every component of a nested tree. 

 

The conversion of the above example into Context-based is quite simple. We don’t need to change the existing component entirely. We only need to create some new components (provider and consumer).

 

# Step-1: Initialize the Context
 

We are creating a context which we will use to create providers and consumers. 

 

const ContextObject = React.createContext(defaultValue)
 

createContext() method is used to create a context. We can also pass the default value to the context variable by passing it to the createContext(). 

 

UserContext.js
 

import React from "react";


// this is the equivalent to the createStore method of Redux

const UserContext = React.createContext({});

export default UserContext;
 

When React renders a component that subscribes to this Context object it will read the current context value from the closest matching Provider above it in the tree. The default value of context is only used when a component does not find the matching provider in the tree. 

 

# Step – 2: Create the Provider
 

Creating context gives us two Provider and Consumer Components. We can create Provider and Consumer components and export them so that they are available for the other components to use them.

 

After creating Provider and Consumer, UserContext.js will look like – 

 

import React from "react";


// this is the equivalent to the createStore method of Redux


const UserContext = React.createContext();


// creating Provider and Consumer and exporting them.

export const UserProvider = UserContext.Provider

export const UserConsumer = UserContext.Consumer


export default UserContext;
 

The Provider needs to wrap around the parent component no matter in which level this context is going to be consumed. Now, I will wrap the parent component (App.js) with the Provider and pass the userCurrent user as to the down tree.

 

App.js
 

import React, { Component } from "react";

import ReactDOM from "react-dom";

import UserProfile from "./UserProfile.js";

import { UserProvider } from "./UserContext.js";



class App extends Component {

 constructor(props) {

   super(props);

   this.state = {

     currentUser: {

       name: "Jhon snow",

       address: "USA",

       mobile: "90899034234"

     }

   };

 }


 render() {

   return (

     <UserProvider currentUser={this.state.currentUser}>

       <UserProfile />

     </UserProvider>

   );

 }

}


const rootElement = document.getElementById("root");

ReactDOM.render(<App />, rootElement);
 

Here, we are passing currentUser information as a provider to the nested child. Currently, we are taking currentUser information from the state. We may fetch user information from API and then we can pass it to the child components. We are importing UserProvider from the UserContext.js file and wrapping the userProfile component inside the provider. Any nested component will have access to the currentUser object.

 

All the consumers that are nested to Provider will re-render whenever the provider’s value changes. 

 

# Step – 3: Create the Consumer
 

We have already created the consumer in the UserContext.js file. The simple way to access the context values is by wrapping the child component in the Consumer for Class component and for the functional component we can access context with the help of useContext method of React. From there, we can access the context value as props. 

 

From our previous example (without using context one), 

 

UserProfile.js
 

import React from "react";

import CurrentUserProfile from "./CurrentUserProfile.js";


const UserProfile = props => {

 return (

   <div>

     <CurrentUserProfile />

   </div>

 );

};



export default UserProfile;


CurrentUserProfile.js (access the Context value.)



import React, { useContext } from "react";


//importing UserContext from UserContext.js file

import { UserContext } from "./UserContext.js";



const CurrentUserProfile = () => {

 const user = useContext(UserContext);


 return (

   <div>

     <p>{user.name}</p>

     <p>{user.address}</p>

     <p>{user.mobile}</p>

   </div>

 );

};


export default CurrentUserProfile;
 

Here, we can see, the currentUserProfile component can directly access the consumer. We don’t need to pass the props manually to every level of the tree. 

 

In the Functional component, we are accessing context with the help of the useContext method.

 

Note. We have the same way of Providing Context to functional components as well as to class components. But in the case of the Consuming context, it is slightly different for function and class components.

Above we have seen Consuming context in functional components. Below is the implementation of CurrentUserProfile.js component using class and consuming context inside it.

 

import React, { Component } from "react";

import { UserConsumer } from "./UserContext";


class CurrentUserProfile extends Component {

 render() {

   return (

     <UserConsumer>

       {props => {

         return (

           <div>

             <p>{props.name}</p>

             <p>{props.address}</p>

             <p>{props.mobile}</p>

           </div>

         );

       }}

     </UserConsumer>

   );

 }

}


export default CurrentUserProfile;
 

As mentioned above, for the class component, we have wrapped the component inside UserConsumer. And we are accessing context as props.

 

What about lifecycle methods? What if we need the value from Context outside of render? The wrapper method was limited. Instead, we can do this in a class with contextType, which is a static variable on the class

 

ContextType:
 

The ContextType property on a class component can be assigned a Context object created by React.createContext() method. This property lets you consume the nearest current value of the context using this.context. We can access this.context in any lifecycle method including the render functions also.

 

Example of Consuming Context in the lifeCycle method.

 

import React, { Component } from "react";

import UserContext from "./UserContext";


class ContextConsumingInLifeCycle extends Component {

 static contextType = UserContext;


 componentDidMount() {

   const user = this.context;

   console.log(user); // { name: "Jhon snow", address: "USA", mobile: ""90899034234"" }

 }


 componentDidUpdate() {

   const user = this.context;

   /* ... */

 }

 componentWillUnmount() {

   const user = this.context;

   /* ... */

 }


 render() {

   const user = this.context;

   /* render something based on the value of user */

   return null;

 }

}


export default ContextConsumingInLifeCycle;
 

Conclusion:
We have successfully learned the React Context API. How to use them with functional components and class components. In the process, we have learned.

Creating Context Const contextObj = React.createContext();
Access contextObj.Provider and contextObj.Consumer from contextObj.
Accessing Context in-class component by wrapping the component inside Consumer.
Access Context in functional component by using useContext method of React.
Consuming Context in the lifecycle method in class components using contextType.
To Know more about react development
<a href="https://www.cronj.com/react/react-development-services">react development services</a>
<a href="https://www.cronj.com/react/react-development-services">offshore react development </a>
<a href="https://www.cronj.com/react/react-development-services">react software development services</a>
