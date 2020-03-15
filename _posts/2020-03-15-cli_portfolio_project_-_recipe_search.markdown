---
layout: post
title:      "CLI Portfolio Project - Recipe Search"
date:       2020-03-15 23:19:37 +0000
permalink:  cli_portfolio_project_-_recipe_search
---


After wrapping up all of our lessons on Ruby, we were tasked with creating a CLI application where a user can make choices to pull information from an external source. It seems like a simple task, but after getting into the weeds a bit, I realized just how involved a simple seeming project like this can be. 

After considering scraping and API as methods to access data, I felt like an API was the best route to go for this type of project. Although scraping allows a broader range of websites to be accessed, APIs are specifically formatted to allow access to data.  My next step was to come up with a broad idea for the project and find an API. I am passionate about plants, so I found and API to a plant database, and I was hoping to create a program where a user can enter a common name for their plant and choose from different information on it. Once I started playing around with the API, though, I realized that most queries for plants contained arrays that were mostly empty. This experience made me realize that if you do go the API route, you should always make sure the API you plan to use has reliable information.

After I landed on a recipe-based API, I got to work on how I wanted the program to operate. There are a lot of different data points stored on each recipe, from as broad as the ingredient list to as specific as the quantity of Zinc, so I chose a few that I thought were most relevant to recipe searching and stuck with those. My idea was to allow users to enter key words to search by, optionally select health restrictions, and optionally set a calorie limit. Then, the program would present some recipe options and allow the user to select one and view information on it. 

My first attempt at this ran the program successfully, but it was not coded in a properly Object-Oriented way. I originally was not saving information to an all variable and was reaching to the API query for every piece of information based on the recipe the user chose. This is not an effective method and relies on a lot of external data pulls. My second variation of the code instead now creates a new instance of  an "Ingredients" class based on the user inputs and then passes that instance through a fetch function in the API class. The fetch function then uses the ingredient instance to pull the correct query only once from the API and create a new instance of the "Recipe" class for each of the available recipe options. Each instance of Recipe only stores the data points that are used in the code. The rest of the program runs off accessing the all array from the Recipe class, which is an array of recipe instances. 

I now have a functional program that allows users to search by their chosen parameters to find delicious recipes. I hope this has helped you understand how to broadly approach a CLI project using an API as well as helped you find some new recipes to try! 

Check out the program [here!](https://github.com/haileybongo/project_cli)
