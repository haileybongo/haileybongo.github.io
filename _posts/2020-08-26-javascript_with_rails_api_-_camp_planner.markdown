---
layout: post
title:      "Javascript with Rails API - Camp Planner"
date:       2020-08-27 01:42:34 +0000
permalink:  javascript_with_rails_api_-_camp_planner
---


After wrapping up the Javascript unit, I approached my first real full stack project. This was the first project that felt like we were putting together all sides of a website - which wasn't always an easy process, but ended up being a very exciting one. 

Inspired by some recent personal trips to national parks, I decided to create a planning website that allows you to put together and store all of your camping trip information in one place. I ended up with three models in my Rails API - a User model, a Trip model, and an Item model. The user model is self explanatory - containing a username, password, and can have many trips. The Trip model consists of a Location, Campsite, Arrival Date, Departure Date, and has_one Item. The Item model, meant to encapsulate "items" you would bring with you in a packing list, consists of a Name and a List. 

When the user first loads Camp Planner, they are prompted with the option to Sign Up or Log In. I went with a User log in model for this site so users could save personal trips and access them later on. Once the user logs in, the DOM is updated to present links in the side navigation bar for My Trips, New Trip, Packing Lists, Find a Campsite, and Log Out. 

### Sign Up & Log In
It is pretty clear what these actions are meant to do! I achieved this by creating two forms, and sending Post fetch requests for each. When the information reaches the rails API, a series of authentication actions are taken to either find or create the user and then create a JWT token which is then sent back to the front end, letting the client side know the user is validated and logged in. 


### My Trips 

The My Trips navigation item is meant to display any existing trips the logged in user has saved. When the user clicks My Trips, a fetch request is made to the rails API. Rails finds the logged in user and then finds all of the user's trips via a rails association. This data is then serialized and transformed to JSON and then sent back to the client side. Once my front end receives the correct array of trips, a new Javascript Object is created for each trip. The Dom is then updated with a list of links, one for each trip to see all of the details. When a trip link is clicked, the Javascript object for the Trip and associated object for the Item are displayed on the DOM. 

### New Trip

When New Trip is clicked, a new trip form appears. This is where the user can create and save information for a new trip. Fields for all of the model attributes are available, as well as a select dropdown for each available Item, or in practical terms, each packing list. When the submit button is selected, a new Post fetch request is made, sending all of this information to the rails API. On the back end, the API creates a new Trip, sets the Trip.user to the logged in user, and sets the Trip.list to the list selected from the drop down. Once this new fully formed trip is saved, it is serialized and sent back as JSON data. The received data is then added to the DOM via a link on the My Trips list and the trip information displayed for review. 

### Packing Lists

This navigation item is meant to allow users to view the content of the available packing lists that they can choose to associate with a trip. Also upon initial load of the site, my first fetch request is called to get each instance of the Item model from my rails API. Once the fetch request is completed and I have a JSON array of information set up by my rails serializer for Item, I then iterate through the JSON data and create a corresponding Javascript object for each Item. I chose to do this in the beginning of the site load since these packing lists are set and created on the back end, and will not change with user interaction. Now, when the user clicks "Packing Lists", the DOM is updated with the information from each JS Item Object. Also, when a new Trip object is created, I can easily grab the ID of the Item associated with the new Trip and find the Javascript object for that Item for presentation on the DOM. 

### Find a Campsite

This item was a stretch feature of my project, since I wanted to try out using fetch with an external API in addition to my rails API. When Find a Campsite is selected, a form populates the DOM with an option to pick a  park from a select dropdown. Each park is attached to a park code that is then sent via a fetch request to the National Park Service API. The fetch responds with a JSON array with a list of campsites and many associated attributes for each site, which are them presented on the DOM for the user to look through.


### Log Out
The log out button clears the JWT token from the client side and updates the DOM to remove links to achieve "only logged-in" actions. Clearing the JWT token means that the client side can no longer send this with any additional fetch requests to the API and therefore the user is no longer authenticated until they log in again and a new JWT token is created and sent to the client. 



Overall, this project showed me how complex dealing with two sides of an application can be but also how rewarding it is to create something that you can see being useful. I have a basic camping trip planner now, but I can imagine a number of features to expand upon, such as the ability to select the campsite you want from the search and add it directly to a new trip form. There are a lot of exciting possibilities for a Javascript with Rails API project, and Camp Planner has given me a sense of that!
