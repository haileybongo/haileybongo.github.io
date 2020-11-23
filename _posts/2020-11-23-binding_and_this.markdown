---
layout: post
title:      "Binding and "this""
date:       2020-11-23 15:32:06 -0500
permalink:  binding_and_this
---


One tricky concept when learning programming is the concept of "this". It can sometimes be difficult to determine the context of "this" and what it is referring to within functions and within our applications. Examining where the function is called can be very helpful in determining "this". Here are the basics:

* When JavaScript function is called, it has its own associated object of an execution context that can be accessed by keyword "this"
* When "this" is used inside a function, it is representing the execution context
* "This" can either be implicit, meaning the code does not specifically say what "this" is, or it can be explicit, meaning the code specifically tells the function what context "this" should refer to


### Implicit Binding
Implicit binding happens when dot notation is used to call a function. Whatever is to the left of the dot when it is called is the  provided context of the function. Let's break down the code below:

```

let hailey = {
name:  "Hailey,
sayName: function (){
console.log(`My Name is ${this.name}`)
    }
}


hailey.sayName()

*Output*
My Name is Hailey


```

In this example, I am creating an object called 'hailey', which contains a name and a function called sayName. Inside the sayName function, I am making use of "this". After I establish my object, I then call the sayName function on the object 'hailey'. Since 'hailey' is to the left of the dot, it implicitly becomes the execution context and is referred to when "this" is used in the sayName function. 

As a general rule, if this is used implicitly, the context can be determined by looking at what is to the left of the dot where the function that contains this was invoked. 

### Explicit Binding
In some cases, we may want to explicitly tell a function what context we want it called on. The first option is to use .call() or .apply() on a function to tell it exactly what we want passed in. For example:


```

let hailey = {
name:  "Hailey

}

function sayName(){
console.log(`My Name is ${this.name}`)
    }


sayName.call(hailey)

*Output*
My Name is Hailey


```

Both .call() and .apply() take a the first argument passed and make it the execution context, or the "this" inside the function that is being called. 

If we want a more permanent explicit solution, we can use .bind(). Bind creates a copy of the original function with the execution context locked in, so whenever the copy is called, "this" is pre-defined. For example:

```

let hailey = {
name:  "Hailey

}

function sayName(){
console.log(`My Name is ${this.name}`)
    }


let haileyGreeting = sayName.bind(hailey)

haileyGreeting()

*Output*
My Name is Hailey


```


### "this" and ES6
Something to be mindful of when using "this" is what notation the function is being written in. in ES6, arrow functions allow us to simplify some code. But it is important to remember that arrow functions are automatically bound to it’s parent's context and do not have their own "this" context, as do traditional functions. When used inside an arrow function, "this" refers to the enclosing scope.  Example below:

```

let greeting = {

type: "Hello",
names: [
"Hailey",
"Lisa",
"Rich",
"Greg" 
]}

let sayHello = function () {
this.names.forEach(name => console.log (`${this.type} ${name}`))
}

sayHello.call(greeting)


*Output*
Hello Hailey
Hello Lisa
Hello Rich
Hello Greg

```

Here we can see that our forEach function is using an arrow function within that refers to "this". Rather than "this" referring to the scope of its own function, the forEach, we can see that it is referring to the "this" defined in it's enclosing scope that we called the function on - the greeting object. 



Hopefully this post was helpful in determining the scope of "this" in code. In summary, it is best to look at where the function is being called to determine if it is implicit binding, explicit binding, or used in an arrow function. 

