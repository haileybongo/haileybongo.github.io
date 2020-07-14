---
layout: post
title:      "Rails Plant Journal: Challenges Tackled"
date:       2020-07-14 17:32:04 -0400
permalink:  rails_plant_journal_challenges_tackled
---


To take on the task of my first rails site, I chose to reinvent a concept I started with Sinatra with a number of added features.My project is a journal application and open source plant information tracker. Users can browse existing plants and watering plans, create new ones, and save plants to their journal. User's journals include information about what plants they have as well as what plants need to be watered. In this post, I will go over some basic details as well as highlight some main parts of this application that were tricky to get up and running!

To begin, I mapped out my plan of data models along with active record associations. My plan was to have four models - User (site users), Journal (journal entries), Plant (types of plants), and Water (watering plans based on plant families). A User has many journal entries, has many Plants that they have created, and has many Waters that they have created. A Plant belongs to a user, has many journal entries written about it, and has many watering plans associated with it through those entries. A Water is essentially the same as a Plant - it belongs to a user, has many journal entries written about it, and has many plants associated with it through journals. 

![](https://drive.google.com/file/d/1SfMkFmzRIsgWlR0g7U_-hieIgKzK3mRZ/view?usp=sharing)

An interesting feature of this site is the new Journal form. This form is rendered through a nested resource (journals are nested within users) through the '/users/:id/journals/new' path. It also incorporates nested forms for the Plant and Water, and allows the selection of an existing one or creation of a new one. To get this going, I added the accepts_nested_attributes_for class method to my Journals model for Plants as well as for Waters.

```
class Journal < ApplicationRecord
    belongs_to :user
    belongs_to :plant
    belongs_to :water
    accepts_nested_attributes_for :plant, :water
```


 When I began to build my form, I knew I wanted to allow users to have the option to select from a drop down of existing plants and watering plans that other users have created. I created two collection_select items that pull each existing plant and water instance and display their names for selection. These items will file to the :plant_id or :water_id attributes of the Journal model. I then wanted to provide the option of creating an entirely new plant or watering plan instead of selecting from a drop down. This is where my nested forms came in - my controller creates a new Plant and Water instance each time the form is rendered and has a form for each, but is only saved if data is entered and there is not a selection on the drop down. Here is an example of the Plant nested form within a journal form:

```

  <fieldset>
      <label><h3> Select or Create Plant </h3></label>
      <%=f.label :plant_id, "Existing Plants to Choose From:" %>
      <%= f.collection_select :plant_id, Plant.all, :id, :name, :include_blank => true%> </br>
  
      <%= f.fields_for :plant, @journal.plant do |plant|%>
        <%= plant.hidden_field :user_id, :value => session[:user_id]%>
        <h5> Or Create a new plant </h5>
          <%= plant.label :name %>
          <%= plant.text_field :name %> </br>
  
          <%= plant.label :characteristics %>
          <%= plant.text_field :characteristics %> </br>
  
          <%= plant.label :light,  "Light Preferences" %>
          <%= plant.text_field :light %> </br>
  
          <%= plant.label :difficulty, "Difficulty Level (1-5)" %>
          <%= plant.number_field :difficulty, min: "0", max: "5" %> </br>
      <%end%>

  </fieldset>
```

Within this form, you can see that the new plant instance is already associated with the journal entry and it contains a hidden field to insert the logged in user as the owner of the plant. With the form complete, I was on to actually taking the data and creating the new objects. 

This is where the params got a little tricky; in order to have the option to select from a drop down or create a new item on creation of a journal entry, I needed conditional parameters. To achieve this, I checked the parameters within the controller for the existence of information in the plant_id key and water_id key. If there was information in one of those, I removed the corresponding plant or water attributes key as to not attempt to create a new instance from them. If there was not information in the plant_id or water_id key, I then removed those from params, letting the plant or water attributes through to create a new instance and automatically associate it with the journal being created. This is how my conditional parameters ended up:



```

def journal_params
        pid = params[:journal][:plant_id]
        wid = params[:journal][:water_id]

        if pid != "" && wid != ""
            params[:journal].delete :plant_attributes
            params[:journal].delete :water_attributes
            params.require(:journal).permit(:date, :last_watered, :user_id, :plant_id, :water_id)
        
        elsif wid != ""
            params[:journal].delete :plant_id
            params[:journal].delete :water_attributes
            params.require(:journal).permit(:date, :last_watered, :user_id, :water_id, plant_attributes: [:plant, :user_id, :name, :characteristics, :light, :difficulty])    
        
        elsif pid != ""
            params[:journal].delete :water_id
            params[:journal].delete :plant_attributes
            params.require(:journal).permit(:date, :last_watered, :user_id, :plant_id, water_attributes: [:water, :user_id, :plant_family, :weeks, :soil])
        
        else
            params[:journal].delete :plant_id
            params[:journal].delete :water_id
            params.require(:journal).permit(:date, :last_watered, :user_id, plant_attributes: [:plant, :user_id, :name, :characteristics, :light, :difficulty], water_attributes: [:plant, :user_id, :plant_family, :weeks, :soil])
        end
      end

```

Although lengthy, this set up allows the optional selection or creation through a nested form in rails! After this challenge was complete, I created standard new forms for both Plant and Water to allow for creation of those on their own. 

My next feature to tackle was the ability to display plants that need to be watered based on user input in journal entries for the date the plant was last watered and the weeks between watering attribute in the associated watering plan. I set this method in the User model, because I wanted to check each journal entry associated with the logged in user and display the plants that needed watering in the User show page. To do this, I first set an empty array needs_water and then iterate through each of the journal entries associated with the current user. If the journal entry has a watering plan associated that has the weeks between watering attribute filled as well as has the last watered date filled out, I then set a new variable days_since_watered equal to today's date minus the last watered date:

```
 def needs_water

        needs_water = []
        self.journals.each do |journal|
          if journal.water.weeks && journal.last_watered
            days_since_watered = Date.today - journal.last_watered

```

At this point, I had a number of days since each of this user's plants listed in journal entries have been watered for each specific entry. If the days since watering was larger than the number of weeks (converted to days) between waterings, the journal entry is added to the needs_water array:

```

if journal.water.weeks * 7 < days_since_watered
                needs_water << journal
            else
            end

```

The end of this method returns the array of journal entries containing plants to be watered. In the User's show page, I then iterate through each of the entries in this array displaying the plants as needing to be watered! The last part of this feature that was a little tricky was creating a button next to each plant to "water" it, or update the date last watered to today.

I achieved this feature by first creating a model method to update this within the journal model. The instance method update_water simply updates the given journal entry's last_watered attribute to today. In the journal controller, I created a method called water that calls update_water on the current journal and then redirects back to the user show page where the button is located. To get to this method, I created a "get" route for '/water/:id' where :id is the journal id that routes to this new water path in the journal controller. The button is set up as such, where "p" is each journal entry in the needs_water array:

```
<%= button_to "Water Plant", "/water/#{p.id}", method: :get%>
```

Once redirected back to the user's home page, the journal entry's last watered date has been updated to today and the entry is no longer in the needs_water array!

There were many other challenges and trials that came with creating my first rails app, but overall I am very proud of this project! I hope the few topics I delved into here shed light on how to handle unique features you want to include in your app and demonstrate that they are achievable with a little bit of work (and binding.pry usage)!


