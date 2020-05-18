---
layout: post
title:      "Complete Sinatra Project"
date:       2020-05-18 02:04:25 +0000
permalink:  complete_sinatra_project
---


At the end of our Sinatra lessons, we were tasked with creating a website using Sinatra and ActiveRecord. Personally, I am very interested in plants and their care, so I came up with the idea to create a plant tracking module that will allow users to add information about their plants as well as their unique watering needs. This site will also allow users to view any plant that has been added on the site, so it can be used to view care information for plants that others have added. Below I will outline how I went about this process, highlighting some interesting pieces of the build. 

My first step after planning was to run the ruby gem Corneal. I did this by completing these three steps:

      * Install the gem with gem install corneal
      * Generate your app with corneal new APP-NAME
      * Run bundle

This left me with a complete Sinatra skeleton! My app was able to successfully connect to a new database and run locally via the shotgun gem. The structure right after running Corneal was as follows:

```
├── config.ru
├── Gemfile
├── Gemfile.lock
├── Rakefile
├── README
├── app
│   ├── controllers
│   │   └── application_controller.rb
│   ├── models
│   └── views
│       ├── layout.erb
│       └── welcome.erb
├── config
│   ├── initializers
│   └── environment.rb
├── db
│   └── migrate
├── lib
│   └── .gitkeep
└── public
|   ├── images
|   ├── javascripts
|   └── stylesheets
|       └── main.css
└── spec
    ├── application_controller_spec.rb
    └── spec_helper.rb
``` 
		

As you can see, Corneal builds an app folder with only an application controller and no models. My next step was deciding what models I wanted to include in my app. I knew I needed a User model along with a Plants model--additionally, I decided to create a third "Water" model to hold optional watering schedule information. 

Corneal has additional functionality to allow you to automatically generate a controller and model with pre-existing views by running 

```
corneal scaffold NAME
```

Where "NAME" is the name of your model. This results in a MVC structure- Models, Views, Controllers. After running this for my User, Plant, and Water models, my structure looked like this:

```
└─app
  ├── controllers
  │   ├──application_controller.rb
  │   └──user_controller.rb
	│   └──plant_controller.rb
	│   └──water_controller.rb
  ├── models
  │   └──user.rb
	│   └──plant.rb
	│   └──water.rb
  └── views
      ├──users
      │  ├──index.html.rb.erb
      │  ├──show.html.rb.erb
      │  ├──new.html.rb.erb
      │  └──edit.html.rb.erb
			 ├──plants
      │  ├──index.html.rb.erb
      │  ├──show.html.rb.erb
      │  ├──new.html.rb.erb
      │  └──edit.html.rb.erb
			 ├──waters
      │  ├──index.html.rb.erb
      │  ├──show.html.rb.erb
      │  ├──new.html.rb.erb
      │  └──edit.html.rb.erb
      ├── layout.erb
      └── welcome.erb
```

My last step before building any core user functionality was creating the database migrations for the users, plants, and waters tables by running rake db:create_migration NAME= "name".  Once the migrations were created, I filled each out with the columns I wanted to store. Here is my plants migration, for example:

```
class CreatePlants < ActiveRecord::Migration[4.2]
  def change
    create_table :plants do |t|
      t.integer :user_id
      t.string :name
      t.string :notes
      t.string :water_id
      t.string :light
      t.timestamps null: false
    end
  end
end
```

Once my environment was set up and connected to a database, my tables were created and migrated, and my app was up and running with a complete MVC structure, I was ready to get going on user functionality!

I started with my user model. Each user must have a secure password, so I added "has_secure_password" to the class. Each user must also have a unique email and username to avoid duplicates, so I added validation for the uniqueness of both the email and username columns within the class. Lastly, each user should have the ability to create many plants, so I added a "has many" relationship with plants. This is as follows:


```
class User < ActiveRecord::Base
    has_secure_password
    has_many :plants
    validates :email, uniqueness: true
    validates :username, uniqueness: true
end

```


The plants and waters models were a bit simpler. since they don't require a password or validation for uniqueness. I only needed to include a has_many :waters and belongs_to :users lines in the plant model and a belongs_to :plants line in the water model. Once the models were set up properly, I moved on to setting up controllers and views.

The users_controller is used to allow visitors to my site to sign up and log in. The  get "/signup"  function renders the signup view containing a form, as long as the current user is not already logged in. Once the sign up form is completed, the "post /signup" function first attempts to create a user. If the new user was created, the username isn't blank, and the email isn't blank, the controller sets the session user_id as the new user's ID and redirects the user to the home page. If this was not successful, the controller collects the error messages in an array and directs the user back to the signup page where the errors are displayed. 

```
post "/signup" do
    @user = User.create(params)
    if @user.save && @user.username != "" && @user.email != ""
      session[:user_id] = @user.id
      @user = current_user 
      redirect "/home"
    else
      @errors = @user.errors.full_messages
      erb :'users/new.html'
    end
  end
```

Other functions within this controller include logging in, logging out, and displaying all plants associated to that user. 

The plants and waters controllers were similar since they both belong to another model. Users should be able to CRUD - Create, Read, Update, and Delete each of these models.  Both controllers contain functions to be able to create a new Plant/Water schedule via the "new" view containing a forms that post and create the new instance as long as the parameters are not blank. These controllers also contain the ability to delete the instance after validating that the logged in user is the user that created the object.  Lastly, each controller also contains the ability to display an individual item within the model. 

One interesting portion of the CRUD model is the "Update" portion. The Edit view contains a display of the current plant, as long as a form to update the plant (and a link to delete the instance as noted above):

```
<html>
 <body>
    <h1>Edit your plant</h1>
    <h3>Current Information:</h3>

        <ul>
            <li><p> Name: <%= @plant.name %></p> </li>
            <li><p> Light: <%= @plant.light%></p> </li>
            <li><p> Notes: <%= @plant.notes %></p> </li>
            </ul>
    <h3>New Information:</h3>
  
    <form method = "POST" action = patch "/plants/:id">
    Name :<input type="text" name="name"><br>
    Light: <input type="text" name="light"><br>
    Notes: <input type="text" name="notes"><br>
     <input id="hidden" type="hidden" name="_method" value="PATCH">
    <input type = "submit" id = "submit">
   </form>

   <form method="post" action="/plants/<%= @plant.id%>/delete">
    <input id="hidden" type="hidden" name="_method" value="DELETE">
    <input type="submit" value="Delete Plant">

 </body>
</html>
```

I wanted to allow users the ability to update just one part of the plant or watering schedule without overwriting the parts they did not update as blank when they submitted the form. I went about this in the  patch "/plants/:id/patch"  portion of the controller by first checking that the params aren't blank, and then checking each key for content before updating that item in the table. 

```

  patch "/plants/:id/patch" do
    if params["plant"] != ""
      @plant = current_plant(params[:id])
      if params[:name] != ""
        @plant.name = params["name"]
        @plant.save
      end
      if params[:light] != ""
        @plant.light = params["light"]
        @plant.save
      end
      if params[:notes] != ""
        @plant.notes = params["notes"]
        @plant.save
      end
      redirect "/plants/#{@plant.id}"
    else   
        redirect "/plants/#{params[:id]}/edit"
    end
  end
```

With my views and controllers built out, I was ready to go! Users can now interact fully with my Plant Tracker app to create new plants, add watering schedules, see all of their plants, delete their plants and watering schedules, and view plant information added by other users. I hope this information was a helpful resource on building out a basic Sinatra application. 
