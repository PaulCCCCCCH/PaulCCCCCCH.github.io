---
title: On Encapsulating HTTP Requests with React Hooks
date: 2022-06-20 13:00:00 -0500
author: Chonghan Chen
categories: [Software Development, Frontend]
tags: [notes, experience, tutorials, software engineering, react, frontend, english]
math: false
---

# Introduction

## The Position of Hooks in the Big Picture 

In my opinion, the key goal of Scaffolding in software engineering is to simplify two things: development and maintenance. A great way to achieve this goal is through **logic sharing**. By sharing logic, you write less code during development, and locate bugs more quickly during maintenance. 

This is the direction to which frontend development world is evolving. With the introduction of frontend frameworks, people are allowed to **define and reuse DOM elements**. Also, we are able to **use pre-defined and shared data-binding and rendering logic**, which all main-stream frontend frameworks provide. Recently, logic sharing is pushed even further. For example, `Vue3.js` introduced Composition APIs to allow grouping functions handling the same data objects together, ignorant of which component is using them, and `React` introduced react `Hooks` to allow easier sharing of logic around certain states. 

For this post, we will focus on `React` and `Hooks`. 

## My Understanding of Hooks

From my understanding, React `Hooks` allows a more flexible way of handling `state`. `state` has been around since `React` was born, but the **handling** (e.g., updating and rendering) of a state seems to be privately owned by a component, which cannot be shared easily. Imagine you have two components, each having some states and hanlder functions for that state. Both components use `stateA`.

```html
Component X {
    stateA, 
    stateB, 
    handlersForStateA, // can be 1000 lines of code
    handlersForStateB,
    
    render: 
    <div>
        <DOM1 state={stateA} />
    	<DOM2 state={stateB} />
    </div>
}
   	
Component Y {
	stateA,
	stateC,
	handlersForStateA,	// can be 1000 lines of code
    handlersForStateC,
            
    render:
	<div>
		<DOM1 state={stateA} />
		<DOM2 state={stateC} />
	</div>
}

```

How would you share the logic here? Without Hooks, you **have to share logic with a helper component**. The helper component looks like this

```html
Component Helper {
	stateA,
	handlersForStateA,

	render:
		<DOM1 state={stateA} />
}
```

Then, you can simply change component X and Y to

```html
Component X {
    // No need for state A here anymore, may save 1000 lines of code
	stateB,
	handlersForStateB,
        
	render:
	<div>
		<Helper />
		<DOM2 state={stateB} />
	</div>
}

Component Y {
    // No need for state A here anymore, may save 1000 lines of code
	stateC,
	handlersForStateC,
        
	render:
	<div>
		<Helper />
		<DOM2 state={stateC} />
	</div>
}
```

Thus, we extracted logic for stateA and freed ourselves from repeating that 1000 lines of code. However, what if we want  to render stateA differently for component X and Y? For example, I want to wrap stateA in a popup in component X, and want to wrap stateB in a table in component Y. How do I achieve this? The answer is, there is no elegant way of doing this other than creating more helper components for encapsulation.

```html
Component PopupHelper{
	render:
		<Popup>
			<Helper />
		</Popup>
}

Component TableHelper{
	render:
		<Table>
			<Helper />
		</Table>
}


```

Then, we use `PopupHelper` in `Component X` and `TableHelper` in `Component Y`. This is frustrating, because **what we really want to share the state part (state + handlers), but we are forced to create a shared component (state + handlers + rendering)**. 

This is where `Hooks` comes into rescue. A hook defines a state (or a set of states) and the logic regarding that state. It is useful to think of it as a component without a rendering logic. With hooks, you can easily decouple state logic with view, thus allowing more flexible logic sharing.

`Hooks` is a powerful tool for logic sharing. In this post, we investigate specifically how we encapsulate the logic around sending HTTP requests to allow the most sharing in the most elegant way possible.  



# Problem Setup

## Request with State 

All HTTP requests need to read from several states, for example, authentication state, frontend side throttling state, and server root url (suppose the user can select which backend to talk to). We assume all state information is aggregated to one object called `apiState` can be read from one `useApiState` call.

## HTTP Request Library

Imagine we have a library to help us send HTTP request, which we can use as

```javascript
lib.get(url: string, state: object)	// For Get requests
lib.post(url: string, data: object, state: object)	// For POST requests
```

Both functions return a `Promise` object, and you can use `await` or `.then()` to resolve. We omit other request types as they are not very different from these two. 

## RESTful API Endpoints

We assume the endpoints follow `RESTful` API styles. For illustration purposes, we consider the following three endpoints we need to cover

1. Type 1, URL only, e.g. `GET https://example.com/user`
2. Type 2, URL with path parameters, e.g. `GET https://example.com/user?first=paul&last=chen`
3. Type 3, URL with path parameters and request body, e.g. `POST https://example.com/user` with `data = {first: paul, last: chen}`

## Goal

The goal is to encapsulate HTTP requests, so that we don't need to worry about all the HTTP intricacies in the components. For example, if I want to fetch user data from the server, I should be able to call them like normal functions, e.g.

```javascript
const users = await fetch_user(param1, param2);
```

or this

```javascript
let users
fetch_user(param1, param2, 
	(res) => users = res.data,   // on success
	(err) => users = [] 		// on error
)
```

or anything equivalent. I shouldn't be assembling request url / headers, etc in my component. I omitted the handling of loading and error state and other things that do not add complexity to our design.

# A Naive Approach (Pure Javascript)

I used to work mostly with `Vue.js` and this solution is what I used first. This approach is framework-agnostic. All implementations are based on pure JavaScript. 

I first create a generic requester for each request types

```javascript
const sendGETRequest(url) {
    const apiState = getApiState() // Problem: how do we get state here?
	return lib.get(`${apiState.server_root}/${url}`, apiState)
}

const sendPOSTRequest(url, requestBody) {
    const apiState = getApiState() // Problem: how do we get state here?
	return lib.get(`${apiState.server_root}${url}`, requestBody, apiState)
}
```

Then, we encapsulate all API endpoints

```javascript
const APIGetUsers() {
	return sendGetRequest("/user")
}

const APIGetUser(first, last) {
	return sendGetRequest(`/user?first=${first}&last=${last}`)
}

const APIPostUser(first, last) {
	return sendPostRequest(`/user`, {first, last})
}
```

Then, we can make these calls anywhere in our program, as if they are plain old helper functions. The problem is, how do we get the `apiState` here? We cannot use `useApiState`, because these functions are neither functional components nor react hooks. We could read from `localStorage` or `Redux` storage, but this means you cannot control the scope of the state (they are always global). For example, it becomes impossible for you to use two different `apiState` in two components. Plus, this does not feel React at all.

# What Most Bloggers Suggest (w/ Hooks)

As I moved from `Vue.js` to `React.js`, I was amazed by `Hooks`. It reminds me of the good old days when I was using `Haskell` , where a huge amount of states were chained together and worked like a miracle (yes, sadly I have to admit it is the feeling of functional programming that attracts me, and `React`.js is not really more elegant than `vue.js` in my opinion). Anyway, I feel the urge to explore a better, more "reacty" way of encapsulating HTTP requests. Here is what most bloggers suggest (you can see a bunch of them by searching for `react useApi hook`).

```react
const useApi(url) {
    const apiState = useApiState() 
    
    // use `result` state to record server's response for this request
    const [result, setResult] = useState() 
    
    useEffect(() => {
        lib.get(`${apiState.server_root}/${url}`)
            .then((res) => setResult(res.data))
        	.catch((err) => setResult(null))
    }, [url])
    
    return [result]
}
```

In a component, we can do the following

```jsx
function SomeComponent {
    const [data] = useApi('/user')
    return <DOM1 state={data}>
}
```

With this approach, we can encapsulate all HTTP request related logic in the `useApi` call. This looks nice, but it is to restricted because:

- It does not support path parameters, and there is no way to change the `url` of  the request. Suppose we want to call `https://example.com/user?first=paul&last=chen` with `first` and `last` parameters from input boxes, this approach simply can't do this. For the same reason, POST requests with a dynamic `requestBody` are not supported.
- By defining `useApi` this way, we explicitly assume the purpose of this request is to retrieve some data and stored in `result` state. However, some requests may require an additional callback (e.g., jump to another page, change another variable), which is not supported by this design.
- The request is sent at loading time. There is no way to manually sending the request again (e.g., when the user clicks a refresh button)
- `setResult` is not exposed to the outside world. The only way to update the state is through sending a request and getting a new result. If you want to allow changing the state to trigger a re-render of a page, you have to use ugly hacks.

Sadly, the top 10 tutorials I found on encapsulating HTTP calls with `Hooks` all stopped here. Even the established libraries (e.g., [`react-use-api`](https://npmjs.com/package/react-use-api)) fail to address all of the problems above. This keeps me thinking, isn't this scenario so common that there should have been a very good solution out there? 



# Building on Solutions From Others (w/ Hooks)

To address the issues mentioned above, I tweaked (if not reformulated entirely) previous solution. Here is a list of things I did.

## Allow Triggering Any Time

Instead of wrapping `lib.get()` in `useEffect` call, I wrap it in a function and expose it to the caller of `useApi`. This allows triggering the request any time.

```react
const useApi(url) {
    const apiState = useApiState() 
    const [result, setResult] = useState()
    
    const sendRequest = () => { // Changed from useEffect to this to allow triggering any time
        lib.get(`${apiState.server_root}/${url}`)
            .then((res) => setResult(res.data))
        	.catch((err) => setResult(null))
    }
    
    return [result, sendRequest]
}
```

## Allow Dynamic URL and Data

We accept an `url` parameter at the time of sending the request, instead of calling the hook. This allows sending requests with dynamic parameters decided at run time.

  ```react
  const useApi() { // removed url parameter here
      const apiState = useApiState() 
      const [result, setResult] = useState()
      
      // put url parameter here so that url can be dynamically changed at send time
      const sendRequest = (url) => { 
          lib.get(`${apiState.server_root}/${url}`)
              .then((res) => setResult(res.data))
          	.catch((err) => setResult(null))
      }
      
      return [result, sendRequest]
  }
  ```

## Do Not Define State Inside

As we mentioned above, defining a result state inside makes the `useApi` hooks much less generic. Instead, we require the caller of `sendRequest` to provide callback functions, which allows the most customization. If the caller wants to define a state, it can still define it in a component, and update it in the callback. This logic can also be shared across multiple components easily just like other logic.

```react
const useApi() {
    // we no long need `result` state
    const apiState = useApiState() 
    
    // accept success and fail callbacks here
    const sendRequest = (url, onSucc, onFail) => { 
        lib.get(`${apiState.server_root}/${url}`)
            .then((res) => onSucc(res))
        	.catch((err) => onFail(err))
    }
    
    // we no longer expose a `result` state
    return [sendRequest]
}
```

## Supporting Multiple Request Types

We can easily extend this template to support multiple request types. 

```react
const useApi() {
    const apiState = useApiState() 
    
    // accept an additional requestType and (optional) requestBody parameter
    // send different request accordingly
    const sendRequest = (url, requestType, onSucc, onFail, requestBody) => { 
        let promise;
        switch requestType {
            case GET:
                promise = lib.get(`${apiState.server_root}/${url}`)
                break;
            case POST:
                promise = lib.post(`${apiState.server_root}/${url}`, requestBody)
                break;
            case DELETE:
                // ... all other types
        }
        promise.then((res) => onSucc(res))
        	.catch((err) => onFail(err))
    }

    return [sendRequest]
}
```



## Additional Encapsulation For Each End Point, Good To Go!

The `useApi` is done. We now need to consider how to use `useApi`. The caller (i.e., the components) will need to provide parameters to `sendRequest` function:

```javascript
const sendRequest = (url, requestType, onSucc, onFail, requestBody) => { ... }
```

Recall that our goal is to **make the components ignorant of the intricacies of HTTP requests**, so we don't want to do this in the component logic

```javascript
const [sendRequest] = useApi()
const url = `/user?first=${userInputFirst}&last=${userInputLast}` // construct endpoint url
const sendRequest(url, GET, ...)
```

This to me is too ugly, because you are hard-coding the request URL template inside a component. You probably need to hard-code it multiple times if multiple components use this endpoint. Thus, let's do an encapsulation for this endpoint.

```react
const useAPIGetUser(onSucc, onFail) {
	const [sendRequest] = useApi()
	const sendGetUsersRequest = (first, last) => {
        const url = `/user?first=${first}&last=${last}`
        sendRequest(url, GET, onSucc, onFail)
    }
    return [sendGetUsersRequest]
}
```

 Then, in any component, all we need to do is

```javascript
const [sendRequest] = useAPIGetUser(onSucc, onFail) // define callbacks
sendRequest(userInputFirst, userInputLast) // send the request
```

There is no HTTP Request related stuff here. All the component knows is that it calls some function and updates its states with callbacks defined within the component. We can simply define one `useAPIxxx` hook for each API endpoint, and put them all in one folder, so that everything related to sending requests will be there.

# Closing Remarks

This has been the most elegant and clean way to encapsulate HTTP calls that I can think of (after all the mind struggles and research). However, it does lead to what's known as callback hell: the component is filled with arrow functions (because of the excessive use of callbacks), and the logic is a bit weird to follow. We could easily change our template to `async/await` style of sending requests: you can just return the result of the inner most `lib.get` out layer by layer instead of calling `then` on it. Personally I think it's OK to leave it as callbacks as it encourages me not to write nested code, which is usually an indicator of bad code design.

To implement `useApiState`, you can use different methods to store different states. For example, use login status may be stored with React `Context API` and `LocalStorage`, and throttling information can be handled with a state manager such as `Redux`.



 







