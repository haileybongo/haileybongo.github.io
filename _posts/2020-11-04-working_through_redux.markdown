---
layout: post
title:      "Working Through Redux"
date:       2020-11-04 07:04:13 +0000
permalink:  working_through_redux
---


After having learned many web development concepts and frameworks, I thought tackling Redux would be par for the course. Once I started my lessions, I quickly learned that Redux was going to take a little bit of getting used to! If you are like me and are having trouble grasping the fundamental concepts of Redux - don't worry. It gets better. And I am going to try to break it down a bit using my final project, Playlister.

On a very basic level, the reasoning to use Redux is that this format will store your data in a place that takes a bit of work to get to, which in turn, protects it. Once you set up your Redux "store", this will become your source of truth for your front end data. For my Playlister app, my store contains saved playlist information, song information, and search result information. All of this data comes from either my back end or the external Spotify API, so it is not being manipulated by basic user input. 

There are obviously many circumstances where you need to update the store, for example when you are loading your data from the back end. To do this, you must call an action. The action will then define what data gathering or manipulation needs to happen, for example sending a GET fetch request to your rails API. Once this action is complete, the action will tell the reducer what it has done and the reducer will then update the state accordingly.

Let's look at an example. When my React Playlist container loads, I want to gather all of the playlists that have already been created from my rails API back end so I can display them for my users. To do this, I call a fetchPlaylists action within my Playlist Container.

```
class PlaylistContainer extends React.Component {

     componentDidMount () {
             this.props.fetchPlaylists()
     } 
```

This code is saying - when my Playlist container loads, call the fetchPlaylists action. So, what does the fetchPlaylists action look like? In my Actions folder, I have a fetchPlaylists file which is exporting my fetchPlaylists action. This action is sending a fetch request to my back end, and then once the front end recievesd the data, I am specifying the information I want to send to my reducer. 
```

export function fetchPlaylists(){
    return(dispatch) => {
    fetch('http://localhost:3000/api/playlists')
    .then(response => response.json())
    .then(data => {dispatch({
        type: 'FETCH_PLAYLISTS', playlists: data})
        })
    }}
    
```


As you can see, once I get a response from my rails API, I am dispatching information that will then be sent to my reducer. I am sending a type of FETCH_PLAYLISTS, and I'm sending the data I received from my back end through a playlists key. 

Let's now jump to what the reducer will do with this information. Reducers are set to take in two parameters - the existing state, and the action. The action, as we just saw, specifies a type and the data we want to utilize. The reducer looks at the type to determine what kind of state manipulation we want to do with this data we have received. 

```
function playlistReducer(state = [], action)  {
    switch (action.type) {
        case 'FETCH_PLAYLISTS':
            return (
                action.playlists.data
            )
        case 'ADD_PLAYLIST':
            return (
                action.playlists.data
            )
        case 'DELETE_PLAYLIST':
            const idx = state.findIndex(playlist => playlist.id === action.id);
            return [...state.slice(0,idx), ...state.slice(idx+1)]  
        
        default:
            return state;
            }
        }
```

In this example, the type defined in the action is FETCH_PLAYLISTS, so the switch statement will evaluate to true on the first line. Within the first case, we are telling our reducer to return the data we received from the fetch action which will then be our state. An important note here is that I am using the Combine Reducers module from React to define different functions for each data group in my state. This snippet is from my playlist reducer, so whatever is returned here will then be defined as the playlists key in my state. 

Whatever is ultimately returned by the reducer then becomes the store. This data can then be used within the front end app by using the Connect module to connect the store to your components using mapStateToProps. Example below:

```
const mapStateToProps = state => {
    return {
        playlists: state.playlists,
        songs: state.songs
    }
}

export default connect (mapStateToProps, { fetchPlaylists })(PlaylistContainer)
```

Hopefully this outline described a bit how Redux works to store data and how that data can be updated and accessed. With a little practice, these concepts will come together and make sense!

