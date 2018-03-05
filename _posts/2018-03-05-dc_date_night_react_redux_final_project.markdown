---
layout: post
title:      "DC Date Night—React / Redux Final Project"
date:       2018-03-05 20:38:30 +0000
permalink:  dc_date_night_react_redux_final_project
---


Woah… it’s time for my *final* Flatiron School project! It’s definitely bittersweet, but overall I am excited and happy to be graduating! And, in my humble opinion, I saved the best project for last. This is something I plan on continuing to build out moving forward, but it’s at a good initial stopping point right now. Let’s get into it!

***Idea***

First, a little bit about the idea. DC Date Night is really centered on this feeling that… Dating can be pretty stressful. But, when you’re the person who takes on planning the date, it can be even *more* stressful. Whether you’re making a first impression, or planning something special for a longtime significant other, you want the date to be memorable, fun, and unique—all while having a feeling of spontaneity. Yikes…

DC Date Night aims to relieve that stress by planning the date for you. It basically says, “Hey, you just worry about which outfit to wear, and we’ll figure out where you should go.” It does this in two ways, both of which are organized by the different neighborhoods throughout the District of Columbia.
* *Curated Dates*—these are dates put together by local DC experts. These experts might be food critics, mixologists, people in the know about art exhibits and museums, etc. They curate a phenomenal date and our users go along for the ride.
* *Generating Custom Dates*—the second way to plan a date is by generating something custom. The user simply selects the neighborhood they would like to be in, chooses a few activities they want to do throughout the date, and our app generates a custom date, just for them.

Voila, your date is planned. Throw on a nice pair of shoes and go get ‘em!

Now that you get the gist of the problem I am trying to solve, let’s walk through some of the initial views / features I have put together, so far. This web app uses React / Redux on the front-end, with a Rails API back-end. I’ll be focusing on the React side of things, throughout most of this post!

<iframe src="https://giphy.com/embed/ikXcqqlSNH2Mw" width="480" height="358" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/ikXcqqlSNH2Mw"></a></p>


***Homepage***

![](https://i.imgur.com/oDp8z0q.png)

The homepage is constructed with a stateful container component, called `Homepage` (duh!). The `return` statement in its `render` method is composed of two stateless functional components: `DateList` (which is passed `dates` as `props`) and `QuickAbout`. 

I use the `componentDidMount` lifecycle method on `Homepage` to fire the `fetchDates` function in `dateActions.js`, once the component has mounted. This is the action that is responsible for asynchronously fetching dates throughout the app and it takes two parameters: the neighborhood that the user wants to see dates for (or just a blank string, to receive all dates) and the max number of dates it should receive in the response. In the case of the homepage, we set the cap to 20.

```
// Homepage.js
componentDidMount() {
    if (this.state.neighborhood || this.state.neighborhood === "") {
      this.props.fetchDates(this.state.neighborhood, 20);
    }
  }
```

```
// dateActions.js
export function fetchDates(neighborhood, cap) {
  let headers = new Headers();
  headers.append('Content-Type', 'application/json');

  return (dispatch) => {
    const reqNeighborhood = JSON.stringify({
      neighborhood: neighborhood,
      cap: cap
    });

    return fetch('/date_entries/browse', {
      method: 'POST',
      headers: headers,
      body: reqNeighborhood
    })
      .then(response => response.json())
      .then(responseJSON => {
        dispatch({type: 'FETCH_DATES', payload: responseJSON})
      })
  }
}
```

The homepage also has the About section, which is rendered using the presentational component, `QuickAbout`. I implemented the `scrollIntoView` method on the `window` object to gracefully bring the user to some information about the site, when they select “About” from the nav bar.

```
// Homepage.js
render() {
    const { hash } = window.location;

    if (hash === "#about") {
      setTimeout(() => {
        const element = document.getElementById('quick-about');
        const datesLoaded = document.getElementsByClassName('date-card').length > 0;
        if (datesLoaded) {
          element.scrollIntoView({behavior: 'smooth', block: "start", inline: "nearest"});
        }
      }, 250);
    } else {
      window.scrollTo(0,0);
    }
```

![](https://i.imgur.com/jFGimmd.png)

***Browse***

![](https://i.imgur.com/uyTcPCn.png)

Next is the browse feature. This is the view that is responsible for displaying all of the curated date options, sorted by neighborhood. The neighborhood options are collected via the `collectNeighborhoodOptions` function inside `dateActions`, which dispatches an action of `type: ‘COLLECT_NEIGHBORHOOD_OPTIONS’` and a `payload` of the parsed response, received from the Rails API. In the `datesReducer`, this updates the `options` object in the `store`, and these options are rendered via the `NeighborhoodSelect` presentational component in `BrowseDates`. The `componentDidMount` method helps us kick this process off, each time the `BrowseDates` component has successfully mounted.

```
// dateActions.js
export function collectNeighborhoodOptions() {
  return (dispatch) => {
    return fetch('/neighborhoods/options')
      .then(response => response.json())
      .then(responseJSON => {
        let options = responseJSON.map(n => n.name);
        dispatch({type: 'COLLECT_NEIGHBORHOOD_OPTIONS', payload: options})
      })
  }
}
```

Now that a neighborhood is selected, dates for that neighborhood are fetched asynchronously via the `fetchDates` function (which was mentioned earlier). We set the cap parameter to `undefined` in this case, in order to receive ALL of the relevant dates (with no maximum number of dates). This updates the `curatedDates` array in `store` via `FETCH_DATES` in `datesReducer`’s switch statement.

```
// datesReducer.js
export default function datesReducer(state = {
  curatedDates: [],
  customDate: undefined,
  options: {},
  editCuratedDate: undefined
}, action) {
  switch(action.type) {
    case 'FETCH_DATES':
      return Object.assign({}, state, {curatedDates: action.payload})
    case 'COLLECT_NEIGHBORHOOD_OPTIONS':
      return Object.assign({}, state, {
        options: Object.assign({}, state.options, {
          neighborhoods: action.payload
        })
      });
    case 'COLLECT_CATEGORY_OPTIONS':
      return Object.assign({}, state, {
        options: Object.assign({}, state.options, {
          categories: action.payload
        })
      });
    case 'STORE_DATE':
      return Object.assign({} , state, {
        customDate: action.payload
      });
    case 'RESET_CUSTOM_DATE':
      return Object.assign({}, state, {
        customDate: undefined
      })
    case 'SET_EDITCURATEDDATE':
      return Object.assign({}, state, {
        editCuratedDate: action.payload
      })
    case 'CLEAR_EDITCURATEDDATE':
      return Object.assign({}, state, {
        editCuratedDate: undefined
      })
    default:
      return state;
  }
}
```

Finally, these dates are then rendered via the `DateList` component, which is also used in the `Homepage` container component. Both of these stateful components access the dates via `this.props`, which is made possible by the `mapStateToProps` function, as well as `connect` from the `react-redux` module . Similarly, all of the actions that are being called are made available via the `mapDispatchToProps` function.

```
// BrowseDates.js
const mapDispatchToProps = (dispatch) => {
  return bindActionCreators({
    collectNeighborhoodOptions: collectNeighborhoodOptions,
    fetchDates: fetchDates
  }, dispatch);
}

const mapStateToProps = (state) => {
  return {
    neighborhoods: state.dates.options.neighborhoods,
    dates: state.dates.curatedDates
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(BrowseDates);
```

Right now it is just this one date, since this is the development site; but, ideally there would be a lot more options here and one of the features I plan on implementing soon is pagination, to make the number of dates on each page less overwhelming by capping it at 15 or so.

If no curated dates are available for a specific neighborhood, a message appears telling the user so and suggests trying to generate a custom date for that neighborhood, instead. This is done by placing the message in the `errors` key of `BrowseDate`’s component state, and passing that value from the state into the `Errors` component via the `errors` prop. This `Errors` component is used for this exact purpose throughout the app, and conditionally displays errors only when they are present in the containing components state.

***Curated Date Show View***

![](https://i.imgur.com/3vKcPLD.png)

![](https://i.imgur.com/oeWn0Ld.png)

![](https://i.imgur.com/Uho6Glz.png)

If a user clicks on a date in the Homepage or in Browse view, the user is routed to the `ShowDate` component, using `React Router`. This view is really the user’s itinerary for the date. It tells them the neighborhood the date is in, gives some detail about the date as a whole, shows them an image that helps them visualize the neighborhood, and then describes each spot that the couple will go to throughout the date.

`ShowDate` iterates through the spots of the selected date and generates a `SpotCard` component for each one, passing in the props that presentational component needs to render the correct information. This information for each spot includes the name of the spot, an image that was uploaded for that spot (or a default image, made possible via the super handy `Paperclip` gem), and a description of why that particular spot was chosen for this curated date.

This particular date is in the Shaw neighborhood of DC and is for oriented towards beer lovers. They begin at *Right Proper Brewing Company*, then get outside and head to *Dacha Beer Garden*, and finally head to *Flash* for some late night, eclectic electronic music.  

***Curate***

![](https://i.imgur.com/e7mD9Vb.png)

If the user selects “Edit Date” on a date’s show view, or they select “Curate” from the nav bar, they will be routed to the `CuratedDateForm` component. This is the form used for both creating and updating curated dates.

The form has a number of input fields used for collecting information about the new or existing date. This includes fields for: the date’s title, a description for the date, the neighborhood it is in, a cover photos, and the ability to add as many spots for the date as they wish. Each `SpotForm` component (which is used inside the `CuratedDateForm` to collect the information for each spot) has fields for: the name of the spot, the category the spot is in, a description of the spot, and a photo for the spot.

These fields, as with all of the text inputs throughout the app, are controlled form inputs. This means they are directly attached to that component’s state and are updated using a host of functions, such as: `updateInput`, `updateSpotAttributes`, `updateCoverPhoto`, `updateSpotPhoto`, `addAdditionalSpot`, etc.

In cases where there are stateless functional components handling the input of specific information inside container components (`SpotForm` being a good example), these functions to handle updates have been passed in as props and use the `bind` keyword to ensure that `this` continues to reference the container component.

```
// CuratedDateForm.js
generateSpotForms() {
    const categories = this.props.options.categories;

    const spotForms = this.state.spots.map((spot, index) => {
      let uniquePhotoVar = `spotPhotoField${index}`;
      return (
       <SpotForm
         key={index}
         title={spot.title}
         description={spot.description}
         updateSpotAttributes={this.updateSpotAttributes.bind(this)}
         categories={categories}
         index={index}
         selectedCategory={spot.category}
         deleteSpot={this.deleteSpot.bind(this)}
         updateSpotPhoto={this.updateSpotPhoto.bind(this)}
         photoRef={(field => (this[uniquePhotoVar] = field)).bind(this)}/>
      )
    })

    return spotForms;
  }
```

If a user wishes to edit an existing date, the same form is presented but it is auto-filled with the existing information for that date. This is done by firing the `fetchExistingDate` function in `dateActions` and setting the initial state of the `CuratedDateForm` component to the values in the response. Since the form’s fields are controlled, they will immediately reflect the values that are set in this initial state.

A word on next steps for this particular feature. Obviously, this form is something that should only be accessible to logged in users; and, at first, it will probably be further limited to “expert” users who have the authorization to curate dates. So, authentication and authorization are features I will be adding in the very near future. Eventually, the ability to curate dates will be opened to all users who have an account, to make the app more interactive.


***Generate***

![](https://i.imgur.com/xfidCU0.png)

If a user doesn’t find anything that they like in curated dates, or if they just want more control over which activities they do throughout this occasion, they can have a custom date generated for them. This is accomplished using the `GenerateDateForm` component.

First, they select a neighborhood from the `NeighborhoodSelect` component. Once a neighborhood has been selected, `collectCategoryOptions` fetches activity options that are available for that specific neighborhood. This is fired in the callback of `setState` inside the `updateNeighborhood` function, so that we can ensure the neigborhood has been set in `GenerateDateForm`’s state and can be included in the request for available activities.
```
// GenerateDateForm.js
updateNeighborhood = event => {
    this.setState({
      neighborhood: event.target.value,
      activities: []
    }, () => {
      if (this.state.neighborhood !== "") {
        this.props.collectCategoryOptions({neighborhood: this.state.neighborhood})
      };
    });
  }
	
	// dateActions.js
	export function collectCategoryOptions(neighborhood) {
  return (dispatch) => {
    let headers = new Headers();
    headers.append('Content-Type', 'application/json');

    return fetch('/categories/options', {
      method: "POST",
      headers: headers,
      body: JSON.stringify(neighborhood)
    })
      .then(response => response.json())
      .then(responseJSON => {
        let options = responseJSON.map(n => n.name);
        dispatch({type: 'COLLECT_CATEGORY_OPTIONS', payload: options});
      })
  }
}
```

The user can then select as many activities (called `categories` on the backend of the app) as they wish before clicking “Plan My Date”. This “Plan My Date” button is only available if they select both a neighborhood and at least one activity, as that is how appropriate Spots are queried from the database.

![](https://i.imgur.com/4h0mYgs.png)

Once a user has generated their custom date, they are shown the itinerary that they will embark on! The spots that are chosen for their date must be in the neighborhood that the user has selected and must belong to one of the categories that the user has indicated they are interested in. If numerous Spot options are available for the neighborhood and activity combination, one is selected at random; though, in the future, this may be further customized so that the most relevant spots for each individual user are surfaced. They also have a button at the bottom of this view that allows them to go back, if they want to roll the dice again.


***Next Steps***

Part of the reason I loved the idea for DC Date Night is because of all the possibilities for expanding functionality. This is a project that I intend to keep building on and improving, moving forward. One obvious feature I plan on implementing soon is the ability to tap into third party APIs for the generation of custom dates. All of the date spots in the app right now are simply hard coded in; but I’m excited to start pulling from related platforms, such as: Foursquare for food recommendations, Ticketmaster for live music, Eventbrite for various events and city happenings—the possibilities are endless.

As I mentioned before, I am also excited to roll my own authentication for DC Date Night, and in doing so, give users the ability to create and share their own dates that they think other users would enjoy. This will make the platform more interactive—more like a two-way dialogue. 

Since I come from a strategy background, the marketing possibilities also jump out at me. We could forge partnerships with various local influencers, whether they be social media personalities, food writers, cultural trendsetters, etc. These influential figures could share their knowledge and create their own curated dates for our users to try. Similarly, we could implement sponsored curated dates, so local restaurants, cocktail bars, music venues, etc could promote their brand in a unique and relevant way.

Finally, there are a lot of ways I could dive into new programming concepts, by continuing to build out this product. Integrating Google Maps to show the spots geographically, learning React Native to bring this idea to mobile devices in a native medium, using OAuth so users can login via their Facebook (or even Bumble?) account. There is a lot of work to be done, and so many directions it could go in. 


***Thanks!***

So, that’s DC Date Night, up to this point. A simple web app that aims to make dating a little less stressful and a lot more fun—for first dates and 25 year anniversaries alike. Keep checking in to watch it grow and, if you’re ever in the DC area, give one of our dates a try! Just remember to take small bites.

<iframe src="https://giphy.com/embed/jU2ZYjA8ngdKU" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/jU2ZYjA8ngdKU"></a></p>




